## Table of Contents
1. [Docker Basics and Common Commands](#docker-basics-and-common-commands)
2. [Managing Container Images](#managing-container-images)
3. [Running Containers](#running-containers)
4. []()
5. []()
6. []()
7. []()
8. []()
9. []()
10. []()

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

Using the container ID prefix:

```bash
docker stop b31
```

Using the container name:

```bash
docker stop laughing_edison
```


## What Happens Without `-dit`

```bash
docker run debian
docker ps
```

- The container starts and exits immediately.
- No container ID is shown.
- `docker ps` shows nothing running.

To keep it alive:

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

### Step 1: Pull the Image

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

### Step 2: List Images

```bash
docker images

#output:
REPOSITORY   TAG     IMAGE ID       CREATED        SIZE
nginx        latest  e445ab08b2be   2 weeks ago    126MB
```

### Step 3: View Image History

```bash
docker history nginx
```

- Each line = a layer
- Some layer hashes may be missing due to build cache not being local

### Step 4: Show Full Image ID

```bash
docker images --no-trunc
```


## Tagging Images

- Tags provide versioning info.
- Default tag is `latest`.
- Tags are similar to Git tags — they point to a specific version.

### Help Command

```bash
docker tag --help

# output:
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### Create a Custom Tag

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

Sample `Dockerfile`:

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

### Key Dockerfile Instructions

- `FROM`: Base image
- `LABEL`: Metadata (author/contact)
- `ENV`: Set environment variables
- `RUN`: Execute shell commands (build time)
- `COPY`: Copy local files into image
- `EXPOSE`: Declare container listening port
- `CMD`: Container startup command

### Best Practices

- Combine commands using `&&` to reduce layers
- Use `apt-get clean` to remove cached packages
- Avoid unnecessary packages to reduce image size


## Building a Custom Image

Prepare files:

```bash
ls

# output:
Dockerfile  index.html
```

Build the image:

```bash
docker build -t mynginx .
```

- `-t` = tag
- `.` = current directory

Run the container:

```bash
docker run -dit mynginx
docker ps

# output:
CONTAINER ID   IMAGE     COMMAND                   ...   PORTS     NAMES
9eb953fd171d   mynginx   "nginx -g 'daemon of..."  ...   80/tcp    zealous_jennings
```


## Checking Layers and Image Versions

### View Image History

```bash
docker history mynginx
```

### List Images Again

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

Docker treats the software inside a container like a system process.

- If the main process exits, the container stops.
- To keep it running, use the `-d` flag to detach.

Foreground execution is fine for single tasks, like:

- Downloading a file
- Processing and saving data

Most containers run background services. Use `-d` to "daemonize" the container.


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

Work backwards if you're stuck:

- Start from the image name.
- Missing images will cause failure no matter the options.

Common mistake: forgetting `-d`. This causes immediate stop.

If a container with the same name exists, you must remove it before reusing the name.


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