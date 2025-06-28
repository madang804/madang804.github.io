## Table of Contents
1. [Docker Basics and Common Commands](#docker-basics-and-common-commands)
2. [Managing Container Images](#managing-container-images)
3. [Running Containers](#running-containers)
4. [Exposing Containers to the Public Network](#exposing-containers-to-the-public-network)
5. [Connecting to Running Containers and Managing Container Output](#connecting-to-running-containers-and-managing-container-output)
6. [Building Images with Dockerfiles](#building-images-with-dockerfiles)
7. [Managing Docker Volumes](#managing-docker-volumes)
8. [Docker Networking](#docker-networking)

---

# Docker Basics and Common Commands

Docker commands work the same across Linux, macOS, and Windows.

When you run a container:
- If the image already exists, Docker uses it.
- If not, Docker downloads it from Docker Hub.

## Running a Debian Container

```bash
docker run -dit debian

# output:
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
5ae19949497e: Pull complete
Digest: sha256:903779f3...
Status: Downloaded newer image for debian:latest
b317be249c1520d561d673470f11c242fbbc81c774d64bc0dcca79c936373eb2
```

- Docker pulls the image if it's not present.
- Starts a container using that image.
- Outputs a long container ID (used to manage it).

You can use the following to refer to a container:

- A short prefix of the ID (e.g., `b31`)
- Or a human-readable name (e.g., `laughing_edison`)

## Docker Run Options

- `-d`: Detached mode (runs in the background)
- `-i`: Interactive mode (keeps `stdin` open)
- `-t`: Allocates a pseudo-TTY (for terminal access)

Together: `-dit` keeps the container alive in the background with interactive shell access.

## Confirming the Container Is Running

```bash
docker ps

# output:
CONTAINER ID   IMAGE   COMMAND   CREATED        STATUS        PORTS   NAMES
b317be249c15   debian  "bash"    3 minutes ago  Up 3 minutes          laughing_edison
```

Columns:

- CONTAINER ID: First 12 characters of the container's full ID
- IMAGE: The Docker image used
- COMMAND: Default command on start
- STATUS: Runtime duration
- NAMES: Auto-generated or user-specified name

## Stopping a Running Container

#### Using the container ID prefix:

```bash
docker stop b31
```

#### Using the container name:

```bash
docker stop laughing_edison
```

## What Happens Without `-dit`

```bash
docker run debian
docker ps

# output: empty
```

- The container starts and exits immediately.
- No container ID is shown.
- `docker ps` shows nothing running.

#### To keep it alive:

```bash
docker run -dit debian
```

## Listing Downloaded Images

```bash
docker images

# output:
REPOSITORY   TAG     IMAGE ID       CREATED         SIZE
debian       latest  00bf7fdd8baf   4 weeks ago     114MB
```

## Inspecting a Container

```bash
docker inspect 0e0
```

- You can use a unique prefix of the container ID.
- Returns detailed JSON output:
    - Networking
    - Environment variables
    - Hostname
    - Start command

## Getting Help with Docker

```bash
# Top-level help:
docker --help

# Help with a subcommand:
docker images --help

# Help with a nested subcommand:
docker image prune --help
```

---

# Managing Container Images

## Image Layering

- Docker images use a layered filesystem.
- Each layer represents changes made to the image (like revision control).
- Base images (e.g., Debian) can be reused by multiple child images to save space.
- Layers are read-only and act as deltas from the layer below.
- Fewer layers usually mean:
    - Smaller image size
    - Faster builds due to better caching

## Inspecting an Image

#### Step 1: Pull the Image

```bash
docker pull nginx

# output:
Using default tag: latest
latest: Pulling from library/nginx
f5d23c7fed46: Pull complete
918b255d86e5: Pull complete
8c0120a6f561: Pull complete
Digest: sha256:...
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

#### Step 2: List Images

```bash
docker images

#output:
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
nginx        latest  e445ab08b2be   2 weeks ago    126MB
```

#### Step 3: View Image History

```bash
docker history nginx
```

- Each line = a layer
- Some layer hashes may be missing due to build cache not being local

#### Step 4: Show Full Image ID

```bash
docker images --no-trunc

# output:
REPOSITORY TAG    IMAGE ID                                                                CREATED     SIZE
nginx      latest sha256:e445ab08b2be8b178655b714f89e5db9504f67defd5c7408a00bade679a50d44 2 weeks ago 126MB
```

## Tagging Images

- Tags provide versioning info.
- Default tag is `latest`.
- Tags are similar to Git tags — they point to a specific version.

#### Help Command

```bash
docker tag --help

# output:
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

#### Create a Custom Tag

```bash
docker tag nginx:latest nginx:myblog_stable
docker images

# output:
REPOSITORY   TAG             IMAGE ID       CREATED         SIZE
nginx        latest          e445ab08b2be   2 weeks ago     126MB
nginx        myblog_stable   e445ab08b2be   2 weeks ago     126MB
```

- No extra space used — Docker reuses the existing image
- Deleting the original tag does not remove the image

## Finding Dockerfiles and Tags

- Use hub.docker.com to find:
    - Image tags
    - Dockerfiles

## Understanding a Dockerfile

#### Sample `Dockerfile`:

```dockerfile
FROM debian:buster-slim
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION 1.17.2
ENV NJS_VERSION 0.3.3
ENV PKG_RELEASE 1~buster

RUN apt-get update \
    && apt-get install -y nginx \
    && apt-get clean

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

COPY index.html /var/www/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

#### Key Dockerfile Instructions

- `FROM`: Base image
- `LABEL`: Metadata (author/contact)
- `ENV`: Set environment variables
- `RUN`: Execute shell commands (build time)
- `COPY`: Copy local files into image
- `EXPOSE`: Declare container listening port
- `CMD`: Container startup command

#### Best Practices

- Combine commands using `&&` to reduce layers
- Use `apt-get clean` to remove cached packages
- Avoid unnecessary packages to reduce image size

## Building a Custom Image

#### Prepare files:

```bash
ls

# output:
Dockerfile  index.html
```

#### Build the image:

```bash
docker build -t mynginx .
```

- `-t` = tag
- `.` = current directory

#### Run the container:

```bash
docker run -dit mynginx
docker ps

# output:
CONTAINER ID   IMAGE     COMMAND                   ...   PORTS     NAMES
9eb953fd171d   mynginx   "nginx -g 'daemon of..."  ...   80/tcp    zealous_jennings
```

## Checking Layers and Image Versions

#### View Image History

```bash
docker history mynginx
```

#### List Images Again

```bash
docker images

# output:
REPOSITORY        TAG             IMAGE ID       CREATED              SIZE
mynginx           latest          3836b15e210f   About a minute ago   147MB
nginx             latest          e445ab08b2be   2 weeks ago          126MB
nginx             myblog_stable   e445ab08b2be   2 weeks ago          126MB
debian            buster-slim     83ed3c583403   4 weeks ago          69.2MB
```

---

# Running Containers

## Why Containers Stop Immediately

- Docker treats the software inside a container like a system process.
  - If the main process exits, the container stops.
  - To keep it running, use the `-d` flag to detach.

- Foreground execution is fine for single tasks, like:
  - Downloading a file
  - Processing and saving data
- Most containers run background services. Use `-d` to "daemonize" the container.

## Running a Detached Debian Container

```bash
docker run -dit debian

# output:
aefe0007dff8e0f0caaa2f04bde774cfebef7a37112ba9dc24fec29d23326f92
```

- `-d`: detached
- `-i`: interactive
- `-t`: allocate a TTY
- `-dit`: allows an interactive shell in the background

## Naming Containers

Hashes aren't human-friendly. Use `--name` to assign a readable name.

```bash
docker run -dit --name=web debian
```

Now you can refer to it as `web`.

## Listing Containers

```bash
docker ps

# output:
CONTAINER ID   IMAGE    COMMAND   CREATED         STATUS        PORTS   NAMES
1bb3e72c7a64   debian   "bash"    41 seconds ago  Up 40 seconds         web
aefe0007dff8   debian   "bash"    2 minutes ago   Up 2 minutes          nice_buck
```

To see all containers, including stopped ones:

```bash
docker ps -a
```

## Latest Container

```bash
docker run -dit --name=third_container debian
```

To check the latest container:

```bash
docker ps -l

# ouput:
CONTAINER ID   IMAGE    COMMAND   CREATED         STATUS        PORTS   NAMES
Ob91465e53e5   debian   "bash"    13 seconds ago  Up 12 seconds         third_container
```

`-l`: shows the most recently created container (running or not)

## Debugging `docker run`

- Work backwards if you're stuck:
  - Start from the image name.
  - Missing images will cause failure no matter the options.
- Common mistake: forgetting `-d`. This causes immediate stop.
- If a container with the same name exists, you must remove it before reusing the name.

## Stopping and Removing Containers

```bash
docker stop web
docker rm web
```

## Restarting Containers

Start a stopped container:

```bash
docker start <CONTAINER>
```

## Auto-Restart on Failure or Reboot

```bash
docker run -dit --restart=always --name=fourth_container debian
```

Verify:

```bash
docker inspect fourth_container | grep -A3 RestartPolicy
```

Common restart policies:

- `always`
- `on-failure`
- `unless-stopped`

## Getting Help

```bash
docker run --help
```

## Forcing Container Shutdown

Use `docker kill` if `docker stop` fails:

```bash
docker kill <CONTAINER>
```

Warning: `kill` skips graceful shutdown.

## Clean Up Stopped Containers

```bash
docker system prune
```

You’ll be prompted to confirm.

## Auto-Clean with `--rm`

Run a one-off container that deletes itself after stopping:

```bash
docker run --rm hello-world
```

## Viewing Logs from Background Containers

```bash
docker logs <CONTAINER_ID>
```


## Summary of Useful Commands

Task	                | Command
------------------------|-------------------------
Run and detach	        |`docker run -dit <image>`
------------------------|-------------------------
Name a container	    |`--name=<name>`
------------------------|-------------------------
List running	        |`docker ps`
------------------------|-------------------------
List all	            |`docker ps -a`
------------------------|-------------------------
Show latest	            |`docker ps -l`
------------------------|-------------------------
Stop container	        |`docker stop <name>`
------------------------|-------------------------
Remove container	    |`docker rm <name>`
------------------------|-------------------------
Restart on failure	    |`--restart=always`
------------------------|-------------------------
Kill container	        |`docker kill <name>`
------------------------|-------------------------
Clean up	            |`docker system prune`
------------------------|-------------------------
Auto-remove after run   |`docker run --rm <image>`
------------------------|-------------------------
View logs	            |`docker logs <name>`

---

# Exposing Containers to the Public Network

We'll run a container and open a specific port so we can access its web server in a browser.

For command syntax or tips for a specific image, check its page on [Docker Hub](https://hub.docker.com). Prefer official images for safety.


## Running a Container and Exposing a Port

Example using the [nginx official image](https://hub.docker.com/_/nginx):

```bash
docker run --name our_nginx -d -p 8080:80 nginx

# output:
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:latest
<container_id>
```

#### What This Does:

- `-d`: Runs container in detached mode (in the background)
- `-p 8080:80`: Maps port 8080 on the host to port 80 in the container
- `nginx`: Pulls the nginx:latest image if it's not already present

#### Port Mapping:

```bash
docker ps

# output:
CONTAINER ID   IMAGE    COMMAND                 CREATED         STATUS        PORTS                 NAMES
e7c485c54894   nginx   "nginx -g 'daemon of..." 30 seconds ago  Up 28 seconds 0.0.0.0:8080->80/tcp  our_nginx
```

Explanation:

- `8080` = host machine (public)
- `80` = container's internal web server port
- `0.0.0.0:8080->80/tcp` = all interfaces on the host are listening on port 8080 and forwarding to port 80 inside the container

## Accesss the Web Server

#### From local machine:

```bash
curl http://localhost:8080
```

#### Access the Web Server from another device on the network:

Use the Docker host's IP:

```bash
http://192.168.1.83:8080
```

## View Web Access Logs

```bash
docker logs our_nginx
```

- Shows who accessed the server
- Useful for debugging or monitoring


## Stop the Container

```bash
docker stop our_nginx
```

## Serve Your Own Web Content

Create a directory and an `index.html` file:

```bash
mkdir webpages
echo 'Hi from inside the container!' > ./webpages/index.html
```

Run a new container and mount the directory:

```bash
mkdir webpages
docker run -p 8080:80 --name another_nginx -v ${PWD}/webpages:/usr/share/nginx/html:ro -d nginx
```

#### Explanation:

- `${PWD}`: Current directory (bash variable)
- `-v`: Volume mount
    - Format: `host_path:container_path[:options]`
    - `ro`: Read-only inside container

The container now serves your custom HTML file.

## Test It

#### With `curl`:

```bash
curl http://localhost:8080

# output:
Hi from inside the container!
```

---

# Connecting to Running Containers and Managing Container Output

- It's uncommon to SSH into a container.
- Instead, you usually:
  - Adjust the Dockerfile
  - Rebuild the image
  - Redeploy the container
- Still, there are times when you want to interactively inspect a running container.

## Run a Container with Shell Access

```bash
docker run -it --name apache httpd /bin/bash

# output:
root@60457c088849:/usr/local/apache2#
```

This command:
- Uses `-it` for interactive shell
- Skips `-d` to keep the container in the foreground
- Starts with `/bin/bash` as the default command

Example interaction:

```bash
root@60457c088849:/usr/local/apache2# pwd
/usr/local/apache2
root@60457c088849:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs  modules
```

Exit the container using `exit` or `Ctrl+D`.

## Check Container Status

```bash
docker ps
```

No container appears because:
- The container exited when `/bin/bash` ended
- It was not detached (`-d` not used)

## Run Container in Detached Mode

```bash
docker run -dit --name second_apache httpd /bin/bash
```

- Container is running in the background. 
- Note: Running `/bin/bash` in detached mode isn't useful for interaction

## Bash vs. Sh in Containers

- Some images (like Alpine Linux) don’t use bash
- Use `/bin/sh` instead:

```bash
docker exec -it container_name /bin/sh
```

Or you can also use `sh`, as long as `sh` is in the container's `$PATH`.

```bash
docker exec -it container_name sh
```

## Use `docker exec` to Enter Running Containers

```bash
docker run -dit httpd
docker exec -it <container_id_or_name> /bin/bash
```

Without `-it`, the shell will run and immediately exit.

## Use `docker exec` to Run Commands Inside

Start a new container:

```bash
docker run -dit --name execution httpd
```

Create a file inside the container:

```bash
docker exec -d execution touch /root/hello
```

Check from inside the container:

```bash
docker exec -it execution bash

# Inside container:
ls /root

# output:
hello
```

Or directly from host:

```bash
docker exec -it execution ls /root

# output:
hello
```

## Installing Tools Inside Containers

- Many images are minimal and lack common tools.
- You can install packages temporarily in a running container:

```bash
apt update && apt install -y <package>
```

But
- This only affects the current container
- You must build a new image to persist changes

## Viewing Processes Inside a Container

- You might find `ps` missing inside the container.
- Use `docker top` from host instead:

```bash
docker top <container name or id>

# output:
UID     PID   PPID  C STIME TTY   TIME     CMD
root    11673 11646 0 16:31 pts/0 00:00:00 httpd -DFOREGROUND
daemon  11727 11673 0 16:31 pts/0 00:00:00 httpd -DFOREGROUND
```

This avoids needing to install `procps` inside the container.

---

# Docker Registries

Docker registries store and distribute Docker images.

## Key Concepts

- Docker Hub is the default registry.
- A registry holds multiple repositories.
- A repository contains one or more images, each identified by a tag.
- Tags often represent versions or variations (e.g., Alpine vs Debian base images).

## Pulling Images

Use `docker pull` to download images.

```bash
docker pull docker.io/ubuntu:bionic

# output:
bionic: Pulling from library/ubuntu
...
Status: Downloaded newer image for ubuntu:bionic
docker.io/library/ubuntu:bionic
```

Same command, simplified:

```bash
docker pull ubuntu:bionic
```

You can also use other DNS formats:

```bash
docker pull registry.hub.docker.com/library/ubuntu:bionic
```

## Viewing Pulled Images

```bash
docker images

# output:
REPOSITORY                                 TAG      IMAGE ID       CREATED         SIZE
ubuntu                                     bionic   a2a15febcdf3   11 days ago     64.2MB
registry.hub.docker.com/library/ubuntu     bionic   a2a15febcdf3   11 days ago     64.2MB
```

- Same `IMAGE ID` means same image.
- Different repository names affect how you reference them.

## Running Images by Repository

```bash
docker run -dit registry.hub.docker.com/library/ubuntu:bionic
```

This shows that image names follow this format:

```bash
registry_address/repository/image:tag
```

For official images, library/ is implied and usually hidden:

```bash
docker run ubuntu:bionic
```

## Removing Images

Delete a specific image:

```bash
docker rmi ubuntu:bionic
```

Run again:

```bash
docker run ubuntu:bionic
```

Docker will pull it again if not found locally.

## Example: Non-Official Images

```bash
docker pull mysql/mysql-server

# output:
Using default tag: latest
latest: Pulling from mysql/mysql-server
...
Status: Downloaded newer image for mysql/mysql-server:latest
docker.io/mysql/mysql-server:latest
```

- Namespace: `mysql`
- Repository: `mysql-server`
- Tag: `latest` (default)

Another example:

```bash
docker pull docker pull madang804/flask-app:v1

# output:
v1: Pulling from madang804/flask-app
...
Status: Downloaded newer image for madang804/flask-app:v1
docker.io/madang804/flask-app:v1
```

- Namespace: `madang804` (Docker Hub username)
- Repository: `flask-app`
- Tag: `v1`

#### Naming Format

```bash
docker_username/repository:tag
```

## Security Risks

- Official images still contain known vulnerabilities.
- Unofficial images may include malware.
- Be cautious. Isolate or build your own images when possible.


## Other Popular Registries

- Amazon ECR: `*.ecr.amazonaws.com`
- Google GCR: `gcr.io`
- Red Hat Quay: `quay.io`


## Private Registries

- Businesses often use private registries for better control.
- Docker Hub supports private repositories.
- Set up with authentication and access control.

## Logging Into Docker Hub

```bash
docker login
Username: <username>
Password: <password>
Login Succeeded
```

To avoid password warnings:

- Use a credential helper.
- Don’t use `-p` on command line if possible.

### Login Syntax

```bash
docker login -u <username> [registry]
```

If no registry is provided, Docker Hub is used by default.

---

# Building Images with Dockerfiles

Each instruction in a Dockerfile creates a separate layer in the final image.

## Key Dockerfile Instructions

- **FROM**: Defines the base image.
- **CMD**: Sets default command arguments or the default command itself.
- **RUN**: Executes commands during image build.
- **EXPOSE**: Opens network ports.
- **VOLUME**: Declares shared storage locations.
- **COPY**: Copies files from local disk to the image.
- **LABEL**: Adds metadata to the image. (deprecated)
- **ENV**: Defines environment variables.
- **ENTRYPOINT**: Sets the main executable to run when the container starts.

## Simple Dockerfile

```dockerfile
FROM debian:latest
LABEL maintainer="Madan Gurung"
ENTRYPOINT ["/bin/ping"]
CMD ["www.docker.com"]
```

#### Breakdown:

- `FROM debian:latest` Uses the latest Debian image.
- `LABEL` Adds metadata about the author/maintainer.
- `ENTRYPOINT` Runs /bin/ping when the container starts.
- `CMD` Supplies the default argument (www.docker.com) to ping.

Resulting execution: `/bin/ping www.docker.com`

## Build the Image

```bash
docker build -t madang804/dockerping:latest .
```

- `-t` adds a tag/name.
- Format: `dockerhub-username/image-name:tag`
- `.` specifies current directory for Dockerfile.


## View Images

```bash
docker images

# output:
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
madang804/dockerping  latest    bc80449d4a5b   19 seconds ago   114MB
debian                latest    85c4fd36a543   2 weeks ago      114MB
```

## Push to Docker Hub

```bash
docker images

# output:
docker login
docker push madang804/dockerping:latest
```

## Override CMD

```bash
docker run madang804/dockerping:latest

# output:
64 bytes from server-13-33-231-30.lax3.r.cloudfront.net (13.33.231.30): icmp_seq=1 ttl=61 time=78.7 ms
64 bytes from server-13-33-231-30.lax3.r.cloudfront.net (13.33.231.30): icmp_seq=2 ttl=61 time=76.1 ms

# override CMD
docker run madang804/dockerping google.com

# output:
64 bytes from lax17s34-in-f14.1e100.net (172.217.11.78): icmpt_seq=1 ttl=61 time=119 ms
64 bytes from lax17s34-in-f14.1e100.net (172.217.11.78): icmpt_seq=2 ttl=61 time=77.2 ms
```

- First attempt, ping `docker.com`
- Second attempt, ping `google.com`

## Override ENTRYPOINT

```bash
docker run --entrypoint /bin/ls -it madang804/dockerping:latest $PWD
```

- This is one way to work with image with an ENTRYPOINT.
- The better alternative is to build image with `CMD` instruction and avoid the `ENTRYPOINT`.

#### Dockerfile: `Dockerfile-allow-override`

```dockerfile
FROM debian:latest
LABEL maintainer="Madan Gurung"
RUN apt update && \
    apt install -y curl && \
    apt clean
CMD ["/bin/bash"]
```

#### Build the Image

```bash
docker build -t madang804/nettools:allow-override -f Dockerfile-allow-override
```

`-f <Dockerfile>` identifies dockerfile that doesn't conform to standard naming convention `Dockerfile`.

#### Run the Image Overriding Default CMD

```bash
docker run -it madang804/nettools:allow-override curl google.com
```

---

# Managing Docker Volumes

- Docker containers are designed to be:
  - Small
  - Portable
  - Disposable
- Images usually include only what's necessary to run a service.

> Avoid storing important data inside containers.

Use volumes to:
- Persist data across container restarts
- Share data between multiple containers

## Volume Types and Mounting Methods

#### Legacy Syntax

```bash
docker run -v /dbdir:/var/lib/mysql -d mariadb
docker run --volume /dbdir:/var/lib/mysql -d mariadb
```

#### Preferred Syntax (`--mount`)

```bash
docker run -d --name container-name \
  --mount source=volume-name,destination=/container/path image-name
```

## Docker Volume Commands

#### Create a Volume

```bash
docker volume create <volume>
```

#### List Volumes

```bash
docker volume ls
```

#### Delete a Volume

```bash
docker volume rm <volume>
```

## Inspecting a Volume

```bash
docker volume inspect <volume>

# output:
[
  {
    "Name": "<volume>",
    "Mountpoint": "/var/lib/docker/volumes/<volume>/_data",
    "Driver": "local",
    "Scope": "local"
  }
]
```

## Mounting a Volume in a Container

```bash
docker run -d --name withvolume \
  --mount source=mydata1,destination=/root/volume nginx
```

Avoid spaces in `--mount`. You can also use:
- `src` instead of `source`
- `dst` or target instead of `destination`

Inspect the container’s mounts:

```bash
docker inspect withvolume | grep Mounts -A 10

# output:
"Mounts":[
  {
    "Type": "volume",
    "Name": "mydata1",
    "Source": "/var/lib/docker/volumes/mydata1/_data",
    "Destination": "/root/volume",
    ...
  }
```

## Add Data to Volume from Host

```bash
echo 'Hello from the mydata1 volume!' > /var/lib/docker/volumes/mydata1/_data/index.html
```

View it from inside the container:

```bash
docker exec -it withvolume bash
cd /root/volume
cat index.html

# output:
Hello from the mydata1 volume!
```

## Mount a Read-Only Volume

```bash
docker run -d --name readcontainer \
  --mount src=newestvolume,dst=/usr/share/nginx/html,readonly nginx
```

You can also use shorthand `ro` for `readonly`.

### Check `read-only` status:

```bash
docker inspect readcontainer | grep Mounts -A 10

# output:
"Mounts":[
  {
    "Type": "volume",
    "Name": "newestvolume",
    "Source": "/var/lib/docker/volumes/newestvolume/_data",
    "Destination": "/usr/share/nginx/html",
    "RW": false,
    ...
  }
```

### Test inside container:

```bash
docker exec -it readcontainer bash
touch /usr/share/nginx/html/test

# output:
# touch: cannot touch '/usr/share/nginx/html/test': Read-only file system
```

You can grant `read-write` to one container and `read-only` to another using the same volume.

## Ephemeral (tmpfs) Volumes

- Temporary in-memory volumes destroys when the container stops.
- `tmpfs` volumes cannot be shared between containers.

```bash
docker run -dit --name ephemeral \
  --mount type=tmpfs,destination=/root/volume nginx
```

#### Inspect:

```bash
docker inspect ephemeral | grep Mounts -A 10

# output:
"Mounts":[
  {
    "Type": "tmpfs",
    "Source": "",
    "Destination": "/root/volume",
    ...
  }
```

#### Add size limit (e.g. 256MB):

```bash
docker run -dit --name ephemeral2 \
  --mount type=tmpfs,tmpfs-size=256M,dst=/root/volume nginx
```

#### Check tmpfs volume size:

```bash
docker exec -it ephemeral2 df -h

# output:
Filesystem Size  Used  Avail Use% Mounted on
tmpfs      256M  0     256M  0%   /root/volume
...
```

## Remove Docker Volume

Ensure to stop container that has volume attached, before removing the volume.

```bash
docker volume rm <volume>
```

To remove all unused volume:

```bash
docker volume prune
```

---

# Docker Networking

Docker leverages the Linux kernel's `netfilter` system via `iptables`. In newer systems, `nftables` may eventually replace this. Docker adapts to whatever networking mechanism the Linux kernel provides.

## Launching a Basic Web Server with Port Mapping

```bash
docker run -dit -p 8080:80 php:apache
```

This starts a container using the `php:apache` image. This image includes Apache with `mod_php`.

Check the container status:

```bash
docker ps

# output:
CONTAINER ID   IMAGE        COMMAND                PORTS                NAMES
90c1116fcdcf   php:apache   "docker-php-entrypoi"  0.0.0.0:8080->80/tcp pensive_gould
```

## Inspect Docker's iptables Configuration

```bash
iptables -nL DOCKER

# output:
Chain DOCKER (1 references)
target  prot opt source      destination
ACCEPT  tcp  --  0.0.0.0/0   172.17.0.2  tcp dpt:80
```

- Docker assigns a non-routable IP (e.g. `172.17.0.2`) to each container.
- Port 8080 on the host maps to port 80 in the container.

Test internal and external routing:

```bash
curl http://172.17.0.2       # container internal IP
curl http://127.0.0.1:8080   # host IP with mapped port
```

Stop the container:

```bash
docker stop pensive_gould
```

## Host Networking Mode

To run a container using the host's networking stack:

```bash
docker run -dit --network host php:apache
```

Check iptables:

```bash
iptables -nL DOCKER

# output is empty as the container shares the host’s network stack
```

Test:

```bash
curl http://127.0.0.1

# output: access to apache server
```

Use `--network hos`t only when absolutely necessary. It removes network isolation.

## List Docker Networks

```bash
docker network ls

# output:
NETWORK ID     NAME     DRIVER    SCOPE
6ca35f7a7b02   bridge   bridge    local
9b2863b1f2e1   host     host      local
9bf18b382a88   none     null      local
```

- The default `bridge` network is used unless specified otherwise.
- Containers on the same bridge can communicate.

## Inspect a Network

```bash
docker network inspect bridge
```

Launch a container and inspect:

```bash
docker run --name w1 -dit php:apache
docker network inspect bridge

# output:
"Containers": {
  "fbf5ded6...": {
    "Name": "w1",
    "MacAddress": "02:42:ac:11:00:02",
    "IPv4Address": "172.17.0.2/16"
  }
}
```

Launch another:

```bash
docker run --name w2 -dit php:apache
docker network inspect bridge

# output:
"Containers": {
  "fbf5ded6...": {
    "Name": "w2",
    "MacAddress": "02:42:ac:11:00:03",
    "IPv4Address": "172.17.0.3/16"
  }
}
```

Now you'll see both `w1` and `w2` with separate IPs (e.g. `.2` and `.3`).

## Container-to-Container Communication

```bash
docker exec -it w1 bash
curl http://172.17.0.3

# output: HTML return from w2
```

Exit and check logs:

```bash
docker logs w2

# output:
172.17.0.2 - - [date] "GET / HTTP/1.1" 403 "-" "curl/7.64.0"
```

This confirms communication between `w1` and `w2` via the default bridge network.

## Isolated Networks for Application Groups

Create a custom bridge network:

```bash
docker network create blog
```

Create a volume:

```bash
docker volume create blog_web_data
```

Run a container in the `blog` network and attach the `blog_web_data` volume:

```bash
docker run --name web --network blog -p 80:80 \
  --mount src=blog_web_data,dst=/var/www/html \
  -dit php:apache
```

Inspect:

```bash
docker network inspect blog

# output:
"Containers": {
  "153be86f...": {
    "Name": "web",
    "IPv4Address": "172.18.0.2/16"
  }
}
```

Test network isolation:

```bash
docker exec -it w1 bash
curl http://172.18.0.2
```

`curl` should fail since `w1` is not on the `blog` network.

