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

* `docker compose build`: builds Docker images, or rebuilds if `Dockerfile` has changed
* `docker compose run --rm %service_name% <command>`: starts a new container to run a command, then deletes the container
  * `docker compose run --rm %service_name% composer install -o`: executes commands in the service
* `docker compose up -d`: starts services from the images
* `docker compose exec %service_name% <command>`: executes a command in an already running container
  * `docker compose exec %service_name% bash`: opens an interactive shell
* `docker compose down`: stops services
  (container's ephemeral filesystem will be removed, including the files that haven't been persisted in a specific volume)
