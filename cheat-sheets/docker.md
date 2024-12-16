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
