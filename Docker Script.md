[============================================]
[==] 🐳 Dockerfile -> build -> run -> ... [==]

@[Dockerfile]

  # FROM node:<version-tag>

Changing working directory
  # WORKDIR /app

Copy package.json and then install the app. 
If you copied everything in src-code before install that would cause NPM install to run again 
  # COPY package.json /app

  # RUN npm install

Copy everything into /app in the container
  # COPY . /app || COPY . . (second dot points to the WORKDIR)

  # EXPOSE 80
This will serve when container is started not when image is being built

  # CMD ["node", "server.js"]

[*][*][*][*] *QUICKSTART* [*][*][*][*]

*#**#**#*  Couple of CLI commands pt. 1 *#**#**#* 
  # docker [ps] <- (lists all running containers)
  # docker [build] . <- (builds a container from the "Dockerfile")
  # docker [run] -p 3000:80 <id> <- (runs the docker container) (-p stands for PUBLISH) (3000 is local, 80 is container's)
  # docker [stop] <id> <- (stops the docker container)
  # docker [run] -it node <- interactive session (is exposed by using the "-it" flag)
  # docker login <- Gives a possibility to login. Asks for credentials.

*#**#**#*  Important *#**#**#* 
1. Docker has its own network so even if you run "npm run dev" you will not see it on :3000
2. You can't edit container itself, it is created and that's it - changes in your local repo will not be reflected.
3. Docker will try to use CACHE if there are no significant changes made eg. COPY will not run
4. Instructions are a equivalent of LAYERS
5. IMAGE is read-only
6. If one layer has changed all subsequent layers are RUN

  # docker [run] ...params
<- if you run it with a different port eg. 9399 you start ANOTHER CONTAINER (foreground - attatched, you can see console's output) whereas...

  # docker [start] <id>
...can run the existing container (background - detatched) (aka. restart)

  # docker [attach] <id>
- Attach yourself to a container

  # docker [logs] <id>
- Logs the output of the container

*#**#**#*  Interactive mode *#**#**#* 

  # docker run -it <id>
Starts in interactive

  # docker start -a -i <id>
Docker start with --attach and --interactive mode

*#**#**#*  Couple of CLI commands pt. 2 *#**#**#* 

  # docker rm <id> <id> <id>...
If you would want to remove a container or multiple containers you can do that
! They need to be stopped first

  # docker images
Lists images

  # docker rmi <id> <id> <id>...
Removes image(s)
! Image will be removed only if no container is using it (stopped or started)

  # docker image prune
Removes all unused images. Adding -a will remove all images

  # docker run ... --rm ... <id>
Once this container is stopped it will be removed

*#**#**#*  Important stuff *#**#**#* 

Containers will build upon images.
Even if you see SIZES displayed like this it does not mean their size is their added values.


> REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
> none         none      fa58e3e7224a   2 minutes ago   997MB
> node         latest    20c0a0be5115   2 days ago      991MB
> mongo        latest    ee13a1eacac9   2 months ago    696MB

*#**#**#*  Couple of CLI commands pt. 3 *#**#**#* 

  # docker image inspect <id>
Inspects an image. Result is a JSON.

@ Not really a good practice
  # docker cp (folder/file.ext || folder/.) <id>:/path
Copying [local => container]

  # docker cp <id>:/path /localpath
Copying [container => local]

  # docker run ... --name anynameofyourchoice <id>
Naming your container

@[Dockerfile]
  # FROM node:14
This makes image use 14th version of the node image
aka repository:tag <- tag is unique

  # docker build -t REPOSITORY:TAG .
Buils an image with convented naming

  # docker run ... (<id> can be equal to REPOSITORY:TAG)
Builds an image from convented naming

*#**#**#* Sharing images *#**#**#*

You can share [Dockerfile] itself (build step is required) or already Built Image (no build step required)

Images can be pushed into:
(Official Docker Image Registry) [Docker_Hub] <- OR -> [Private_Registry]

  # docker push <name> ...params
Pushing an image.

  # docker pull <name> (by default it goes to [Docker_Hub]) use <host_name:name> instead to pull an image from private Registry
Pulling an image.

  # docker tag (old) REPOSITORY:TAG (new) user/REPOSITORY:TAG
Makes a clone with a new name (This is needed for the instruction below).

Step by step:
1. Create a image repository on [Docker_Hub]
2. Copy the path
3.  # docker push user/hello-world (This will only work if you have the same named repo locally exactly: "user/hello-world")

If you push a tag into a repository it will be displayed.

  # docker pull name/name
Pulls latest image from the targetted registry.

  # docker run <image-name>
Will run a local image but if it can't find it will use [Docker_Hub] (There are no check for the latest version).

[*][*][*][*] *Volumes and Bind Mounts* [*][*][*][*]

1. If you stop a container without a "--rm" option data WON'T BE LOST 🔃
2. But will be LOST if you remove the container.

[I] VOLUMES ARE FOLDER ON YOUR OWN HOST MACHINE.
[I] DOCKER IS AWARE OF THEM AND MAPS THEM INSIDE DOCKER CONTAINER

@Dockerfile (path(s) for persisted data)
  # VOLUME [ "/<workdir>/feedback" ]
⬆ This is a ANONYMOUS volume (location is unknown) (volume only exists when container exist and you
as a developer cannot access it)

@Cli: (adding named volume - are not attached to the container)
  # docker run ... -v feedback:/app/feedback
this represents the path we want to save, before the colon comes name of the volume
Data is now persistent. Try stopping and then running the container again. You will see the data peristed.

  # docker volume rm <name>
Will remove specified volume.

  # docker volume prune
Will remove all volumes. It will ask before.

*#**#**#* Bind Mounts *#**#**#*
kind of Volumes but! you can map the exact location on localhost (dev now sees it)
...are perfect for persisted and editable data.
- it is specific for the CONTAINER not the image.

Named volume can help persisting but EDITING is not possible. Since its location is unknown.


[*][*][*][*] *DEEP DIVE!* [*][*][*][*]
- make sure docker has ACCESS to the folder you're binding to (check it inside Docker Desktop) (eg. /Users/)
  # docker run ... --rm -v feedback:/app/feedback -v "/Users/lukaszjonasiak/Desktop/data-volumes-03-adj-node-code/server.js:/app" <image:tag>

Specify the path in double quotes so that whitespace or special characters are ignored.
KEEP IN MIND THAT IT IS OVERWRITTEN! (we make worthless [Dockerfile] because of the fact of overwriting) (node_modules might be a problem)
At least docker is NOT going to overwrite your local files. (Other way it actually works.)

==========================================
==== You can use shortcuts like these ====

macOS / Linux: -v $(pwd):/app

Windows: -v "%cd%":/app

==========================================

If it comes to bind mounts/volumes the MORE SPECIFIC path wins.

Example of bind mount:

  # docker run ... -v /app/node_modules/ ...
(node modules now wins and previously stated bind mount is not winning with the anonymous volume)

OR

@[Dockerfile]
  # VOLUME ["/app/node_modules"]

...and yes this works like a watcher for the container.

*#**#**#* Nodemon *#**#**#*

Even if you've got a bind mount on your image, changes won't reflect in node-run environment.

You can install a specific package that will make changes in NODE apps reflected. (node file change watcher)

  # devDependency called "nodemon": "^2.0.4"

  # and in "scripts": { "start": "nodemon server.js" }

@[Dockerfile]
  # CMD [ "npm", "start" ] change to => CMD [ "nodemon", "file" ]

If you are using WSL2 you should store it somewhere in the LINUX folder structure? what...

*#**#**#* Summary of Volumes and Bind Mounts *#**#**#*

1. Anonymous Volumes
@[Dockerfile]
  # VOLUME ["/temp"]

CMD:
  # docker run ... -v /app/data/ ...

Characteristics:
- Created for the specific container
- Survives container shutdown / restart until it gets removed or "--rm"...ed
- Data inside it cannot be shared across the containers
- It cannot be reused, even on the same image

=> Useful for not-overwriting things in your bind-mounted project

2. Named volumes
  # docker run ... -v name:/inside_docker/path... ...
Cannot be created in [Dockerfile] since they are not attached to the container itself.

Characteristics:
- Created in general - not tied to any specific container
- Survives things that Anonymous Volumes can and REMOVAL of the container
- Can be removed but with...
- Can be shared across the containers
- Can be reused for the same container

3. Bind Mounts
  # docker run ... "/User/absolute/path/:/inside_docker/path" ...
- LOCATION is actually known. (Which is not possible with VOLUMES)
- Can survive anything!!!
- Only way to remove it, you have to remove it ON YOUR LOCAL MACHINE. (yes) (you can't use docker cmd for this purpose)
- Can be shared across containers
- Can be reused for the same container

*#**#**#* readonly Volumes *#**#**#*

Docker should NOT be able to edit anything on your local machine.
Without the following option docker is able to perform editing in your LOCAL bind mount.

Usage: at the end of the BIND MOUNT statement add ":ro" which stands for read only.

  # docker run ... -v "/Users/lukasz/.../:app:ro"

... but some of the folder must be overwritten on our local system eg. feedback generated files.
(more specific volume overwrites the less specific volume)

so that:

  # docker run ... -v /app/temp

Temporary container's files overwrite and write to our local host machine regardless of the bind mount.
However, this tactic of overwriting only works if you execute it in the CMD!!!.

*#**#**#* Managing Volumes *#**#**#*

  # docker volume ls
lists Active volumes

<!--
CMD LOG (self explanatory which is which)
local     ea06e0f15138bcaa26a94eb05d80f211146ffd4b5b8efb3c270d2622a5e0f1c2
local     eb40f19ca8fb4c0ca24a6bfda524cb8c435f51283d789e72166f62ca172244f2
local     edc98d8f9dd21ef8d41763b9c71240893cd1c4a4af7ee3e9929fb07ab7d173ab
local     edf239e0ba8bf72f416d74a847d9b894e3d99bf83da9a0952f793051838d8c90
local     f7ef4986b84b9816d7bded66f10bf0b7b9b24504ef50d977aca5bb4d94dde67e
local     fa82d0cbcea5e305e699562902e9ed90540df1d9ececf8a8f905f302b3134196
local     feedback
-->

What is more BIND MOUNT will not show up here since it is managed by YOU!
 
  # docker volume create <volume-name>
You can create your own volume.
It will be automatically created if you don't have it already.

  # docker volume rm && docker volume prune
Self explanatory.
Keep in mind that you cannot remove in-use volume.

  # docker volume inspect <volume-name>
<!-- CMD OUTPUT
[
    {
        "CreatedAt": "2022-04-17T18:42:12Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/feedback/_data",
        "Name": "feedback",
        "Options": null,
        "Scope": "local"
    }
]
-->

Mountpoint is not a local path. You won't find it. It is more like a VM.
It is hard to find out where it is stored.

Something Extra:

@[Dockerfile] 

  # -- COPY . . <- You can comment it out but...

BIND MOUNT is DEV thing. ON your server you will NOT run docker container instance with a CMD command.
We simply won't use BIND MOUNT. In production we want to have a SNAPSHOT instead.

*#**#**#* [.dockerignore] <- file *#**#**#*

@[Dockerfile]
  # COPY . .
now copies everything in the folder.

1. make a "[.dockerignore]" file (this specifies what files should not be copied by COPY instruction)

@[.dockerignore] (consider adding this path | files)
<!--
node_modules
.git
Dockerfile 
-->

[*][*][*][*] *ARGuments and ENVironments* [*][*][*][*]

Docker support build-time ARGuments and ENVironment variables.
One note: Using them at the beginning of the [Dockerfile] might be time consuming and useless thereafter.
Use them in a place where they are needed.

- ARG
Available inside of @[Dockerfile] and NOT accessible in CMD or any other application code.
Set on image build (docker build) via --build-arg

- ENV
Available inside of @[Dockerfile] and in APPLICATION code.
Set via ENV in @[Dockerfile] or via --env on docker run.

===
ENV
===

Setting inside:
@[Dockerfile]

  # ENV NAME=VALUE

example:

  # ANY_ACTION ${NAME}

@CMD

Setting internal port with --env command option
  # docker run docker run ... -p 3000:8000 (--env || -e) PORT=8000

^ If you would like to make multiple envs use --env or -e many times.

+ Adding .env file with --env-file command option.
  # docker run ... --env-file ./env

==== WARNING! ====
Do not put secure data inside your [Dockerfile].
Use separate .env file instead.
If you do not do that .env are cooked into the image and everyone can look into them with
  # docker history <image>

===
ARG
===

Build-time arguments (ENVs are run-time arguments)
We can plug into image by using the [Dockerfile] when we build the image without hardcoding them into the [Dockerfile]
You CANNOT use them in the FILE SYSTEM.

Most USEFUL for changing something between built images.

@Dockerfile
  # ARG DEFAULT_PORT=80
  # ${DEFAULT_PORT}

@CMD
  # docker build ... --build-arg DEFAULT_PORT=80
^ Example

*#**#**#* Networking: (Cross-) Container Communication *#**#**#*
[*][*][*][*] *Networking: (Cross-) Container Communication* [*][*][*][*]


(One container should do one separated thing!)

// Kinds of communicaton:
1. WWW (You don't need any special setup in this case)
2. Host Machine
3. Cross-Container Communication (database, back-end anything...) ->
EXAMPLE: Our front-end app might want to talk to our dockerized backend.

1. ^^^ [WWW Method]
Container -> (req) -> WWW

(If app does not crash it does not CRASH the container)
It just works.

2. ^^^ [Host Machine method]
Container -> (req) -> Host Machine


database://     host.docker.internal     :3000/anyEndpoint

Just use this special domain (host.docker.internal) to kind of point to localhost machines.

3. Cross-Container communication

(3rd party usage: docker run mongo)

  # docker container inspect <id> 

<!-- 
    "NetworkSettings": {
        "Bridge": "",
        "SandboxID": "1a2434a51a1dc70a1c3a35a0f9042d1fa1b4dc67c5434ccb3daca83a0f0e1d5c",
        "HairpinMode": false,
        "LinkLocalIPv6Address": "",
        "LinkLocalIPv6PrefixLen": 0,
        "Ports": {
            "27017/tcp": null
        },
        "SandboxKey": "/var/run/docker/netns/1a2434a51a1d",
        "SecondaryIPAddresses": null,
        "SecondaryIPv6Addresses": null,
        "EndpointID": "2ffd22efaaa87b7bc3cabc3ea90f57ea406f54c736ea53a8e032652fdb249469",
        "Gateway": "172.17.0.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAddress": "172.17.0.2"...
-->

under "IPAddress" is your container's IP stated. In this case "172.17.0.2"

Default Mmongodb port: 27017

*x* ||| Container Networks ||| *x*

Creating isolated container with a network
but network will NOT be created if not existing.
  # docker run ... --network <network-id> ...

Creating a NETWORK:
  # docker network create <new-network-id>

Listing them:
  # docker network list

Now your container name can be plugged into a address and will work fine.

<!--
    mongoose.connect(
    `mongodb://mongodb:27017/swfavorites`,
        ...
    );
-->

*WARNING* If you run a container with -p, this is used only if you want to connect to this container FROM OUTSIDE.

|| Drivers ||
default: 'bridge'

  # docker network create --driver bridge my-net

Drivers:

1. host - Isolation between container and host system is removed.
2. overlay - Multiple Docker daemons are able to connect with each other. Only works in deprectated "Swarm"
3. macvlan - You can set custom MAC Address to a container. this address can then be used for communication with
            that container
4. none - networking is disabled
5. Third-party plugins - You can install 3rd party plugins which then may add all kinds of functions.

but "BRIDGE" makes most sense in the vast majority of scenarios.

*#* Revision *#*

If you run Database container make sure to publish the <port>
  # docker run ... -p 27017:27017

Remember about -it flag if used in React app (Interactive flag).
  # docker run ... -it

node_modules might *CRASH* your application up if you copy them to your working directory.
Keep them out with [.dockerignore].

If you want to make:
  # docker run ... (with) --network <network-name>

Make sure your backend uses the name of the container in database url.

*WARNING* If you would like to talk to the backend. Please make sure, once you dockerize it,
you have already published some port so that client side rendered frontend can easily communicate with it.

*INFO* Docker will LOAD volume if already existing.
  # docker run ... -v name:/path/is/this

Not allowing node_modules to be overwritten by using anonymous volume.
  # docker run -v /app/node_modules

Use "nodemon" to watch changes in node.js application.

Using Bind-mount with frontend app it will reload (if your app code supports hot reload).

[*][*][*][*] *Docker Compose* [*][*][*][*]

[I] Docker compose does NOT replace images or containers.
[I] Docker compose does NOT replace Dockefiles.
[I] Docker compose is used mostly for the SAME host.

Two formats are available.

*docker-compose.yaml* OR *docker-compose.yml*

[Docker-compose] consists of keywords. So make sure you typed them the proper way (Indentation matters).
@docker-compose.yaml

version: "3.8"
services:
  mongodb: # [#] Name of the container
    image: 'mongo' # [#] Specifying image
    volumes: # [#] Specifying volumes
      - data:/data/db
    container_name: mongodb # [#] Setting the name of the container.
   #environment: [#] ENV Variables
      # MONGO_INITDB_ROOT_USERNAME: max [#] (1st syntax)
      # - MONGODB_INITDB_ROOT_USERNAME=max [#] (2nd syntax)
      # (3rd syntax -> use file .env)
    env_file: # [#] Relative path to a .env file
      - ./env/mongo.env
  backend:
    build: './backend' # [#] This command will look for the Dockerfile inside that.
    # build:
    #  context: ./backend [#] Can be a path and it will also be a place of build step for the image.
    #  dockerfile: Dockerfile [#] If you named your Dockerfile different way
    #  arg: [#] You can use ARGs in case your image uses ARGs
    #    some-arg: 1
    ports:
      - '80:80'
    volumes:
      - logs:/app/logs
      - ./backend:/app # [#] In this case you can specify a bind mount with a relative NOT absolute path.
      - /app/node_modules # [#] specifying anonymous volumes.
    env_file:
      - ./env/backend.env
    depends_on: # [#] ONLY in docker-compose. On multiple services this is extra useful.
      - mongodb
  frontend:
    build: './frontend'
    ports:
      - '3000:3000'
    volumes:
      - ./frontend/src:/app/src
    stdin_open: true # [#] Letting docker-compose to know that this service need a open-input connection.
    tty: true # [#] For attaching terminal.
    depends_on:
      - backend
volumes: # [#] Make docker aware of named volumes. So on, different containeres can use the same volume.
  # [#] (Anonymous volumes and Bind:Mounts don't have to be specified there)
  data:
  logs:


[I] --rm is default in dcompose
[I] -d is default in dcompose
[I] For key:value pairs you don't need dashes (this creates a yaml object).

*EXTRA* [docker-compose] will create network automatically for the containers!

@CLI
  # docker-compose up

or in detatched mode...
  # docker-compose up -d

to DELETE all containers are networks from using docker-compose up -d
but the following will not remove volumes.
  # docker-compose down

you can use -v flag to remove them too.
  # docker-compose down -v

[I] Docker will generate names of compose-up containers not only by their service names.
[I] Service names ARE THE NAMES THAT CAN BE USED AS REQUEST URLS etc...
[I] Docker-compose will NOT build images every time.

@CLI

Force image re-building.
  # docker-compose up ... --build

[*][*][*][*] *UTILITY CONTAINERS* [*][*][*][*]

Just ENVIRONMENT.

Use case:
You would need to install Node to run npm command, but...

This container if not used with -it flag will IMMEDIATELY stop.
  # docker run node

...but this will not stop immediately: (You can then attach yourself to it)
  # docker run -it -d node

Allows you to run more commands inside of a container. (If you want to provide input add -it flag)
  # Docker exec -it <container-namer> npm init

Running container & npm command.
  # docker run -it <image-name> npm init

[I] alpine <- slim node image.

----------------
ENTRYPOINT

Restricting commands:

@[Dockerfile]

// and then in cli you can APPEND this command.
ENTRYPOINT [ "npm" ]

so that...

  # docker run ... <image> init

= init is now a string appended to the ENTRYPOINT string. Results in (npm init).

@CLI of [docker-compose]

Running just one service from the docker-compose.yaml
  # docker-compose run <service-name>

^ however this does not remove container automatically.

--------
Utility containers are just for running some environment.
