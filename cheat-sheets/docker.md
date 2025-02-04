# Docker cheat sheet

A highly opinionated [docker](https://www.docker.com/) cheat sheet.

## Ubuntu 24.04 installation

Set up Docker's `apt` repository:

```console
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the latest version:

```console
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Add your UNIX user to the docker group:

```console
sudo usermod -a -G docker $USER # use `jenkins` instead of `$USER` if using jenkins
```

Reboot.

Finally verify that the installation is successful by running the `hello-world` image:

```console
sudo docker run hello-world
```

See:

* [install using the repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
* [got permission denied](https://stackoverflow.com/a/48450294/3437428)

## Usage

* `docker build -t <name>`: builds Docker images (or rebuilds if `Dockerfile` has changed)
  * `docker build -t <name> --no-cache`: rebuilds Docker image everytime
* `docker run -it <name> <command>`: starts a new container to run a command (`-it` for interactive purpose)
  * `docker run -it --rm <name> <command>`: starts a new container to run a command, then on exit deletes the container

* `docker compose build`: runs `build` for each services defined in `compose.yaml` config
  * `docker compose build --no-cache`: forces rebuild
* `docker compose up`: runs `build`, `pull` and `run` for each services
  * `docker compose up --detach`: to run services in the brackground
* `docker compose down`: stops services
  (container's ephemeral filesystem will be removed, including the files that haven't been persisted in a specific volume)

## Maintenance

* `docker images`: lists images
  * `docker images --filter dangling=true`: lists untagged / unused images
* `docker container ls`: lists running containers
  * `docker container ls -a`: lists running and stopped containers
* `docker system prune`: removes dangling containers, networks and images
  *  `docker system prune --volumes`: removes dangling containers, networks, volumes and images
* `docker history <image>`: Inspects layers of an image
