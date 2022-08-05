[========================================]
        Deploying Docker Containers
[========================================]

*I* 1. DEV vs PRODUCTION => Everything is standalone *I*

*I* 2. Rather NOT use Bind Mounts in PROD *I*

*I* 3. Multi-container APPS should be split across multiple hosts *I*

*I* 4. PROD image/container should be a single-source-of-truth (COPY > Bind Mounts) *I*

*I* 5. Make PROD containers "A G N O S T I C" *I*

Big Joints are:

- AWS
- Microsoft AZURE
- Google Cloud

----------------------------
[***] Wroking with EC2 [***]

/ Spin up your remote machines. /

1. Create and launch EC2, VPC (Virtual Private Cloud) and security group
2. Configure security group to expose all required ports to WWW
3. Connect to instance (SSH), install Docker and run container

*I* key-pair = connect with SSH (only downloadable once) (do not SHARE!) *I*

*I* SSH = Secure Shell *I*

-----------------------------------------------------
[***] Installing Docker on VM (Virtual Machine) [***]

Update & and using Latest Check
# ❯ sudo yum update -y

AWS Docker installing
# ❯ sudo amazon-linux-extras install docker

Starting a Docker Service
# ❯ sudo service docker start

------------------------------------------------
[***] Pushing our local Image to the Cloud [***]

1. Deploy Source -> build image on the CLOUD (Unnecessary complexity) (non-sense)
2. Deploy Image -> docker run on the CLOUD (Avoid unnecessary remote server work)

@ [.dockerignore]
node_modules
Dockerfile
*.pem

@ [dockerhub] + config

*WARNING* docker build --platform linux/amd64 ... (use on Mac M1)

Logging into [dockerhub] account
# ❯ docker login

=> Create new repository (mechawolf/node-example-1)

# ❯ docker images

1. Tagging (changing name of an image)
# ❯ docker tag <local-image-name> <mechawolf/node-example-1:tagname>

2. Pushing into a remote repository (SAME NAME AS THE LOCAL IMAGE'S NAME)
# ❯ docker push mechawolf/node-example-1:tagname

3. Running container on the remote is the same as the local (AFTER PUSHING)
# ❯ docker run -p --rm -p 80:80 mechawolf/node-example-1

*I* By default EC2 instance is blocked to anything except your SSH client

[Outbound rules] => control (from -> to somewhere else)

-----------------------------------------------------
[***] Managing & Updating the Container / Image [***]

Pulling latest version
# ❯ docker pull <image-name>

*DISADVANTAGES*
- a lot of work
- Trade-offs between control and responsibility

*I*
- Own Remote Machines = Control but Bad Security
- Managed Remote Machines = Less Control but more Security (ECS = elastic container service)

--------------------------------------------------------------------------
[***] AWS ECS - Managed Docker Container Service (No running server) [***]

Overwrite workdir on start.
# ❯ docker run --workdir ...

*I* Task = One remote machine that runs one or more containers.

*I* FARGATE = Servless (request => executed, no request => stopped) (AWS Specific)

*I* Keep in mind that you can dynamically change.

Updating:
Cluster -> default -> Task definition (running one) -> Create new revision

------------------------------------
[***] docker-compose deploying [***]

You can't use docker-compose in deploying containers.

Adding containers to the same task (ECS) then they are guaranteed to run on
the same machine. (localhost inside EC2 instance)

so...
§
*I* Cluster is just a network for your containers

Create Cluster -> + VPC -> Create a Task Definition -> Create Container

Types some commands if you would like to in "Commands" with comma separated syntax

------------------------------------------------
[***] Persisting data with ECS EFS Volumes [***]

EFS = "Elastic Files System"

mongod.lock => You cannot have two tasks using the same volume (EFS) at the same time.
In different words, two databases cannot use one and the same volume at the same time.

[@] Current architecture:

----                     AWS LOAD BALANCER                     ----
AWS ECS +-> ECS TASK +-> (NODE REST && MONGODB +                  )
                                                 (AWS EFS STORAGE)

[@] Not about Databases:

- Scaling and managing can be challanging
- Performance could be bad (also during traffic spikes)
- Taking care about backups and security can be challing

=> so that... Use managed Database service (MongoDB Atlas, AWS RDS [Relational Database System?])

-------------------------------------------------------
[***] Migrating to MongoDB Atlas [***]

- Remember abount changing your backend's database access.
- Authorize your access with .env file that is more specific than Dockerfile's envs.

// Example
mongodb+srv://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}/${process.env.MONGODB_NAME}?retryWrites=true&w=majority

-------------------------------------------------------------
[***] Using MongoDB in production mode && Architectiure [***]

----                     AWS LOAD BALANCER                    ----
AWS ECS +-> ECS TASK +-> (NODE REST && MONGODB Atlas =           )
                                                 (MongoDB)

---------------------
[***] Frontends [***]

in Production use -> "build" so that the server won't start

*WARNING* Don't use dev server since it is unoptimised for prod use.

"start" command might not need "run" in the npm command.
Dockerfile.prod <- multiple Dockerfiles

@ [Dockerfile] (Build-only containers)
================================================================

[*] Giving them a "stage" name
FROM node:14-alpine as build

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 3000

[*] Just building
RUN [ "npm", "run", "build" ]

FROM nginx:stable-alpine as prod

[*] /usr/share/nginx/html <- is a default target for the NGINX
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

[*] Running the App
CMD [ "nginx", "-g", "daemon off" ]

================================================================

=> Multi Stage builds:

*INFO* Every FROM (Dockerfile's) instruction creates a new stage in your Dockerfile.
Even if you use the same image as in the previous step.

...after RUN command and building the image
change the image to nginx.

Use -f option to run custom Dockerfile
# ❯ docker build -f Dockerfile.prod -t mechawolf/image ./

@ Deploying our frontend

STARTUP DEPENDCY ORDERING (First backend -> then frontends)

*WARNING* Make port listeners unique

Create Task Definition to make two different urls.

{*} Production vs Development {*}

process.env.NODE_ENV ('production' vs 'development')

Choosing --target for the Dockerfile stages.
# ❯ docker build --target <stage-name> ...
