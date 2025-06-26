## Table of Contents
1. [Docker Basics and Common Commands](#docker-basics-and-common-commands)

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

Help with a subcommand:

