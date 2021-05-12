[![CD Status](https://img.shields.io/github/workflow/status/Heg0Dmsk/docker-webhook/Build%20And%20Push%20Docker%20Images?label=Continious%20Deployment&style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Last Commit](https://img.shields.io/github/last-commit/Heg0Dmsk/docker-webhook?style=for-the-badge&logoColor=white&logo=github)](https://github.com/Heg0Dmsk/docker-webhook)
[![Pull Requests](https://img.shields.io/github/issues-pr/heg0dmsk/webhook-docker?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Repo Size](https://img.shields.io/github/repo-size/heg0dmsk/webhook-docker?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)
[![Image Size](https://img.shields.io/docker/image-size/heg0dmsk/webhook-docker/latest?style=for-the-badge&logoColor=white&logo=docker)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![Pulls](https://img.shields.io/docker/pulls/heg0dmsk/webhook-docker.svg?style=for-the-badge)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![Version](https://img.shields.io/docker/v/heg0dmsk/webhook-docker?style=for-the-badge)](https://hub.docker.com/r/heg0dmsk/webhook-docker)
[![License](https://img.shields.io/github/license/heg0dmsk/webhook-docker?style=for-the-badge)](https://github.com/Heg0Dmsk/docker-webhook)


A modified version of [`TheCatLady's webhook`](https://github.com/TheCatLady/docker-webhook) docker container based upon [`adnanh's webhook`](https://github.com/adnanh/webhook),   additionally containing the docker cli and docker compose. Befoe using this conatiner, please inform yourself about the advantages and risks of executing docker commands inside a docker conatainer and access the docker host via the Docker socket, for example [here](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) or [here](https://tomgregory.com/running-docker-in-docker-on-windows/).

# Content
<ul >
  <li><a href="#how_to_use" style="margin: 0px;">How to Use</a>
    <ul> 
      <li><a href="#how_to_use_docker_compose" style="margin: 0px;">Docker Compose</a></li>
      <li><a href="#how_to_use_docker_cli" style="margin: 0px;">Docker CLI</a></li>
    </ul>
  </li>
  <li><a href="#how_to_use">How to Use</a>
    <ul> 
      <li><a href="#updating_docker_compose" style="margin: 0px;">Docker Compose</a></li>
      <li><a href="#updating_docker_cli">Docker CLI</a></li>
    </ul>
  </li>
  <li style="margin: 0px;"><a href="#parameters">Parameters</a></li>
  <li style="margin: 0px;"><a href="#configuring_hooks">Configuring Hooks</a></li>  
</ul>

# Content
- [How to Use](#how_to_use)
  - [Docker Compose](#how_to_use_docker_compose)
  - [Docker CLI](#how_to_use_docker_cli)
- [Updating](#updating)
  - [Docker Compose](#updating_docker_compose)
  - [Docker CLI](#updating_docker_cli)
- [Parameters](#parameters)
- [Configuring Hooks](#configuring_hooks)

<a name="how_to_use"></a> 
# How to use

Docker images are available from [Docker Hub](https://hub.docker.com/r/heg0dmsk/webhook-docker) and [GitHub Container Registry (GHCR)](https://github.com/users/heg0dmsk/packages/container/package/webhook-docker).

<a name="how_to_use_docker_compose"></a> 
## Docker Compose (recommended) 

Add the following volume and service definitions to a `docker-compose.yml` file:

```yaml
services:
  webhook:
    image: heg0dmsk/webhook-docker
    container_name: webhook
    command: -verbose -hooks=hooks.json -hotreload
    environment:
      # optional
      - TZ=America/New_York 
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

<a name="how_to_use_docker_cli"></a> 
## Docker CLI

Run the following command to create the container:

```bash
docker run -d \
  --name=webhook \
  -e TZ=America/New_York `#optional` \
  -v /path/to/appdata/config:/config:ro \
  -v /var/run/docker.sock:/var/run/docker.sock `#exposing the docker socket, needed to access the docker host` \
  -p 9000:9000 \
  --restart always \
  heg0dmsk/webhook-docker \
  -verbose -hooks=hooks.json -hotreload
```

<a name="updating"></a> 
# Updating

The process to update the container when a new image is available is dependent on how you set it up initially.

<a name="updating_docker_compose"></a>
## Docker Compose

Run the following commands from the directory containing your `docker-compose.yml` file:

```bash
docker-compose pull webhook
docker-compose up -d
docker image prune
```

<a name="updating_docker_cli"></a> 
## Docker CLI

Run the commands below, followed by your original `docker run` command:

```bash
docker stop webhook
docker rm webhook
docker pull heg0dmsk/webhook-docker
docker image prune
```

<a name="parameters"></a> 
# Parameters

The container image is configured using the following parameters passed at runtime:

| Parameter                                      | Function                                                                                                                                                                                                                                                                                                                                              |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-e TZ=`                                       | [TZ database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) of system time zone; e.g., `America/New_York`                                                                                                                                                                                                                        |
| `-v /path/to/appdata/config:/config:ro`        | Container data directory (mounted as read-only); your JSON/YAML hook definition file should be placed in this folder<br/>(Replace `/path/to/appdata/config` with the desired path on your host)
| `-v /var/run/docker.sock:/var/run/docker.sock` | Exposing the docker socket, needed to access the docker host    |
| `-p 9000:9000`                                 | Expose port `9000`<br/>(Necessary unless only accessing `webhook` via other containers in the same Docker network)                                                                                                                                                                                                                                    |
| `--restart`                                    | Container [restart policy](https://docs.docker.com/engine/reference/run/#restart-policies---restart)<br/>(`always` or `unless-stopped` recommended)                                                                                                                                                                                                   |
| `-verbose -hooks=/config/hooks.yml -hotreload` | [`webhook` parameters](https://github.com/adnanh/webhook/blob/master/docs/Webhook-Parameters.md); replace `hooks.yml` with the name of your JSON/YAML hook definition file, and add/modify/remove arguments to suit your needs<br/>(Can omit if using this exact configuration; otherwise, all parameters must be specified, not just those modified) |

<a name="configuring_hooks"></a> 
# Configuring Hooks

See [`adnanh/webhook`](https://github.com/adnanh/webhook) for documentation on how to define hooks.
