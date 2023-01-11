[![CD Status](https://img.shields.io/github/actions/workflow/status/Heg0Dmsk/docker-webhook/docker-build-and-push.yml?label=Continious%20Deployment&style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Last Commit](https://img.shields.io/github/last-commit/Heg0Dmsk/docker-webhook?style=for-the-badge&logoColor=white&logo=github)](https://github.com/Heg0Dmsk/docker-webhook)
[![Pull Requests](https://img.shields.io/github/issues-pr/heg0dmsk/webhook-docker?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Repo Size](https://img.shields.io/github/repo-size/heg0dmsk/webhook-docker?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Image Size](https://img.shields.io/docker/image-size/heg0dmsk/webhook-docker/latest?style=for-the-badge&logoColor=white&logo=docker)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![Pulls](https://img.shields.io/docker/pulls/heg0dmsk/webhook-docker.svg?style=for-the-badge)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![Version](https://img.shields.io/docker/v/heg0dmsk/webhook-docker?style=for-the-badge)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![License](https://img.shields.io/badge/LICENSE-MIT-blue?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)

A light image of [`adnanh's webhook`](https://github.com/adnanh/webhook) which is able to access the Docker host and control containers.
This image is based upon [`TheCatLady's webhook`](https://github.com/TheCatLady/docker-webhook) docker image. In addition to some minor tweaks this image has access to the docker host and therefore is able to execute Docker commands on the host. This is possible by exposing the docker socket and adding packets for docker compose and the docker cli.

# Table of Contents
- [Security Concerns](#security_concerns)
- [How to Use](#how_to_use)
  - [Docker Compose](#how_to_use_docker_compose)
  - [Updating](#updating)
- [Parameters](#parameters)
- [Configuring Hooks](#configuring_hooks)

<a name="security_concerns"></a> 
# Security Concerns
In order to run docker commands inside the container and actually execute them on the docker host, the docker socket of the host needs to be exposed. While this allows to execute docker commands (e. g. for a CI/CD workflow) triggered by a webhook, it also poses a security risk. The risk involves that a container which has access to the docker socket may be able to get root access to the host. In order to lessen this security threat a Docker Socket Proxy can be used (e. g. [Tecnativa's Docker Socket Proxy](https://github.com/Tecnativa/docker-socket-proxy)).

Also, if you further want to inform yourself about the advantages and risks of executing docker commands inside a docker conatainer and access the docker host via the Docker socket, take a look for example at [this post](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) from Jérôme Petazzoni or [this post](https://tomgregory.com/running-docker-in-docker-on-windows/) from Tom Gregory.

<a name="how_to_use"></a> 
# How to use

Docker images are available from [Docker Hub](https://hub.docker.com/r/heg0dmsk/webhook-docker) and [GitHub Container Registry (GHCR)](https://github.com/users/heg0dmsk/packages/container/package/webhook-docker).

<a name="how_to_use_docker_compose"></a> 
## Docker Compose (example)

Add the following volume and service definitions to a `docker-compose.yml` file:

```yaml
services:
  webhook:
    image: heg0dmsk/webhook-docker
    container_name: webhook
    command: -verbose -hooks=hooks.json -hotreload
        environment:
      - TZ=Europe/Berlin #optional
    volumes:
      - /path/to/appdata/config:/config:ro
      # exposing the docker socket, needed to access the docker host
      - /var/run/docker.sock:/var/run/docker.sock 
    ports:
      - 9000:9000
    restart: always
```

Then, run the following command from the directory containing your `docker-compose.yml` file:

```bash
docker-compose up -d
```

<a name="updating"></a> 
# Updating

The process to update the container when a new image is available is dependent on how you set it up initially. If you initially used Docker Compose, run the following commands from the directory containing your `docker-compose.yml` file: 

```bash
# Pull latest version of the images specified in the docker-compose.yml file
docker-compose pull 

# Redeploy
docker-compose up -d

# Remove old dangling Images
docker image prune
```

<a name="parameters"></a> 
# Parameters

The container image is configured using the following parameters passed at runtime:

| Parameter | Symbol | Example| Description                                                                                                           |
| ----------|--------|--------| ---------------------------------------------------------------------------------------------------------------------------- |
| Volume |-v |`/path/to/appdata/config:/config:ro`        | Container data directory (mounted as read-only); your JSON/YAML hook definition file should be placed in this folder<br/>    (Replace `/path/to/appdata/config` with the desired path on your host)                                                               |
| Volume | -v | `/var/run/docker.sock:/var/run/docker.sock` | Exposing the docker socket, needed to access the docker host                                                      |
| Port |-p |`9000:9000`                        | Expose port `9000`<br/>(Necessary unless only accessing `webhook` via other containers in the same Docker network)           |
| Restart Policy | --restart | recommended `always` | Container [restart policy](https://docs.docker.com/engine/reference/run/#restart-policies---restart) |
| Command | | `-verbose -hooks=/config/hooks.json -hotreload` | [`webhook` parameters](https://github.com/adnanh/webhook/blob/master/docs/Webhook-Parameters.md); replace `hooks.json` with the name of your JSON/YAML hook definition file, and add/modify/remove arguments to suit your needs<br/>(Can omit if using this exact configuration; otherwise, all parameters must be specified, not just those modified)         |
| environment | -e | `TZ=Europe/Berlin` | Specifcies timezone of the container, Look up timezones [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)   |


<a name="configuring_hooks"></a> 
# Configuring Hooks

See [`adnanh/webhook`](https://github.com/adnanh/webhook) for documentation on how to define hooks.
