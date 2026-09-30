note to ai before editing the below :(exaplin everything simplyfied. less text but logical person will understand this . new junior devops friendly. can use graphical representation) delete this text after editing.
# Docker Basics  


`docker run <image:tag>`

`docker pull <image:tag>`

`docker ps`

`docker ps -a`

`docker images`

`docker rmi <image or id or ids>`



`docker run ubuntu sleep 3600 `  #find the container id use "docker ps -a" to find it

`docker exec <containerid from previous> cat /etc/hosts`



`docker run -d kodekloud/simple-webapp`  #detached from the terminal

`docker attach <containerid>` #reattach to terminal



`docker stop <containerids>`



`docker rm <container ids seperated with spaces>`



docker system prune  #removes all stopped containers, all networks not used atleast by one container, all danling #images and all unused build cacahe



\-------------------------------------------------------------------------------------------------------------------



\#by default docker does not listen to stdinput and so for interactive mode



`docker run -it kodekloud/simple-prompt-docker`

\# `-i alone to pipe data into the container` # `-t flag is used to show colored output / progress bars (pretty-formatting)`


### port mapping



\#port mapping or port publishing



\#port docker container to a free port on the docker host 



`docker run -p 80:5000 kodeloud/webapp`  # 80 is host port and 5000 is container port





## Volumes

 volumes for persisting of data (map a directory from the host to a directory inside the container)



`docker run -v /opt/datadir:/var/lib/mysql mysql` 



`\#` volume mounts, bind mounts and tmpfs mount



• Volumes: Docker abstracts storage inside /var/lib/docker/volumes/ to provide fully managed, cross-platform persistence isolated from host OS mutations.
``
            `• docker run -d --mount type=volume,source=v\_name,target=/app/data nginx`



• Bind Mounts: Explicitly maps any user-specified absolute path from the host file system directly into the container namespace, bypassing Docker management.
``
            • `docker run -d --mount type=bind,source=/host/path,target=/app/data nginx`



• tmpfs Mounts: Mounts a temporary, volatile file system directly into host memory (RAM), completely avoiding non-volatile disk I/O operations.
``
           • `docker run -d --mount type=tmpfs,destination=/app/cache nginx`







### inspect and logs



`docker inspect <containder name/ id>`



`docker logs <containder name/ id>`



# Dockerfile example


## Docker file:-

`FROM ubuntu`			                    
                                                
`RUN apt-get update`				                
`RUN apt-get install -y python3-flask`		    
                                                
`COPY app.py /opt/app.py`				            
                                                
`ENV FLASK\_APP=/opt/app.py`			            
`ENTRYPOINT flask run --host=0.0.0.0`	       



docker build -t adithyan/my-app .  # -t is the name/tag and the . represent the Dockerfile in the current dir

docker push adithyan/my-app


docker history `<image\_name>`

docker build -t webapp .

#check this github project -  <https://github.com/ksadithyan/simple-flask-app-container-image-builder.git>

MAJOR ISSUES: with the traditional docker builder
1) REDOWNLOADING PACKAGES EVERY BUILD
2) SECRETS LEAK INTO METADATA 
   1) env (data gets baked onto the build history)
   2) copy + rm (data gets baked onto the build history)
   3) --build-arg (data gets baked onto the build history)
   4) multi-stage builds (better but risky so not preferable)
3) ARCHITECTURE LOCK-IN
   1) bad fix - seperate build machine
   2) bad fix - qmeu emulation by hand
4) INDEPENDENT STAGES RUN SEQUENTIALLY


# BUILDKIT
A MORDERN APPROACH THAT DEALS WITH THE ABOVE ISSUES

### docker buildx build -t adithyan/web-app .

use these as fixes for the above major issues

1) `--mount=type=cache,target=/root/.cache/<for example,pip> <the command u need to run>` This caches the files and sppeds up the building
2) `--mount=type=secrets,id=mykey <whatever the command u need to run>`
   1) when u build, `docker buildx build --secret id=mykey,src./key.txt -t myapp`  if you have the key in the key.txt file in the same folder (this pass secret from the outside and no trace)
3) fixes for architecture lock in by using `docker buildx build --platform linux/amd64,linux/arm64 -t ksadithyan/webapp` (can give any proper name)
   1) finish the above code with `--push` to push it to the registry under a single name as a multi-arch manifest
4) buidx by default executes the indepndent pieces side by side with 0 additional effort

### docker CMD and ENTRYPOINT
also mention both use like 
```dockerfile
ENTRYPOINT["sleep"]
CMD[5]
```
explanation needed for the above~ not yet written
### basics of env variables in production use
(write here what are the best practices simple enogh to understand complicated concepts easily)


## Docker Compose
   `include the compose.yaml, init, .dockerignore, other files`
   `profiles,depends_on, health-checkup-depends-on-condition`
   `other details if i've missed`



## Docker Network
   `docker network create <network-name> , docker network ls` `other details`
   `docker run ubuntu --network=`x ; where x can be `none`,`host` and *by default it is bridge if u dont specify --network*
   if using host note that container mapped to host so port entries are fixed i,e both should be same
   ### isolate another network?
      Then use
      ```bash
      docker network create --driver bridge --subnet 182.18.0.0/16 your-isolated-netwrk-name
      ```
### example voting without docker compose

use this repo - <https://github.com/dockersamples/example-voting-app.git>

`docker build -t <xyz>` where xyz can be `./vote`, `./result`, `./worker` 
`docker network create voting-app`
`docker run -d --name vote -p 8080:80 --network voting-app vote `
`docker run -d --name result -p 8081:80 --network voting-app result `
`docker run -d --name worker --network voting-app worker `
`docker run -d --name redis --network voting-app redis:alpine`
```bash
docker run -d --name db --network voting-app \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=postgres \
-v db-data:/var/lib/postgresql/data \
postgres:15-alpine
```

### example voting with docker compose

using this same repo - <https://github.com/dockersamples/example-voting-app.git>

docker-compose-simple.yml
```yaml
services:
  vote:
    image: vote
    ports:
      - "8080:80" 

  result:
    image: result
    ports:
      - "8081:80"

  worker:
    image: worker
  
  redis:
    image: redis:alpine
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

# NOTE: you can also use the build and then location to the Dockerfile to build, 
# but since i have the images already built i'm just using the image in the compose. 
# also you can use the .env file - 
#-------------------------------------.env----------------------------------------------------
# POSTGRES_USER=postgres
# POSTGRES_PASSWORD=postgres
#---------------------------------------------------------------------------------------------
# then use the below instead of the "environment: " in the docker compose.
#----------------------------------------yaml edit--------------------------------------------
# env_file:
#   - .env
#---------------------------------------------------------------------------------------------
```

`docker compose -f docker-compose-simple.yml up` 
the above command also create a network automatically along with the containers

`docker compose -f docker-compose-simple.yml up`
to stop

# Remote docker
example using tls
`docker -H=10.123.2.1:2376 --tlsverify run nginx`

don't use the port 2375 plain tcp - unencrypted and unsafe

best way use ssh://user@host  - this uses ssh

## Underneath Daemon

1) docker - CLI
2) dockerd - Daemon
3) containerd - Runtime
4) runc - OCI

### Namespace
container pid mapped to the host's own ordered pid 
no had isolation

### cgroups - Resource limits
`dockr run --cpu=0.5 ubuntu`
`dockr run --memory=100m ubuntu`

## Docker Storage

`/var/lib/docker` -dir contents - `containers/`, `image/`, `volumes/`, `overlay2/`


# Docker Registry

name/app:version
for private registry - privatregitrywhateveritis.io/xyz/appname:version
- always login bere pushing or pulling to private registy

Runing ur own registry? see below

1) setting up the *local private registry*
`docker -d -p 5000:5000 --name registry registry:2`
2) image tag
`docker image tag my-image localhost:5000/my-image`
3) push
`docker push localhost:5000/my-image`
4) pull from anywhere-within-this-network using *localhost*(if in the same host) or by *ip or domain name* of my docker host if i'm accessing it from another host environment
`docker pull 192.168.56.100:5000/my-image`


Note: always mind the pull rate limits and authenticate when u can


# DOCKER ON WINDOWS
Dockr desktop is only a developer workstation product; default mode is linux containers; swithc mode to windows containers if needed to run windows application/ windows containers. 2 types windows server container and Hyper-v isolation - one shares kernel(like linux) while the other has it's own so better security and different-kernel-version can coexist. The FROM in the Dockerfile changes to Windows Server Core (full fledged heavy) or Nano Server(alpine equivalant - ie, small size)

use the engine only install for windows server 2019,2022,2025

![alt text](image-1.png)