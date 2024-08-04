# nightscout-docker-simple
Simple Nightscout CGM setup in docker on a VPS

This is a simple nightscout docker setup that I use on my VPS running Ubuntu 22.04 in combination with [NginxProxyManager/nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager).

It is inspired by [LostOnTheLine/Nightscout_Docker-Compose](https://github.com/LostOnTheLine/Nightscout_Docker-Compose) and [dhermanns/rpi-nightscout](https://github.com/dhermanns/rpi-nightscout).

I'm running the nightscout docker container as root alongside other containers without any problems.

!Important: I have switched off the alarms by default in the docker-compose.yml. If this is something you rely on make sure to turn it back on.

## Guide

### Step 1 - Connect to your VPS and update system

Open a terminal on your computer and ssh into the VPS.
```
ssh root@<IP_OF_YOUR_VPS>
```

First we’ll update the system.

```
sudo apt-get update && sudo apt-get upgrade
```

### Step 2 - Install Docker

As per [docs.docker.com](https://docs.docker.com/engine/install/ubuntu/):

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

1. Set up Docker's apt repository.
```
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
2. Install the Docker packages.

To install the latest version, run:
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Verify that the Docker Engine installation is successful by running the hello-world image.
```
sudo docker run hello-world
```

### Step 3 - Install and configure nginx proxy manager




Navigate back to ~/apps and create a new directory for your nightscout app, this is where your nightscout will live, including the mongodb

```
mkdir nightscout
cd nightscout
nano docker-compose.yml
```
copy paste the content of docker-compose.yml from this repository. Make sure to change the API Secret key to something at least 12 characters long. You can edit the plugins here too (careportal, iob etc..) For a comprehensive overview see the [nightscout](https://nightscout.github.io/) documentation.

Now its time to fire up our nightscout docker container

```
docker compose up -d
```
If everything went well you can reach your nightscout now in the browser on the direct IP of your VPS on port 1337. e.g. 182.229.239.135:1337

Continue with the last bit of [this guide](https://medium.com/@jmpinney/multiple-wordpress-sites-on-one-server-with-docker-6fb53adc4bfe) to generate SSL certificates for your site


### Stop, edit and restart your configuration

To stop, edit and start your nightscout configuration again first run
```
docker ps
```
Note down the CONTAINER ID of nightscout and monogdb

In order to stop the containers run

```
docker stop <CONTAINER ID DOCKER>
docker stop <CONTAINER ID MONGODB>
```

Now you can make changes to your docker-compose.yml and rerun the following command inside the ~/apps/nightscout directory to start it again.

```
docker compose up -d --remove-orphans
```

### Migrate or reinstall VPS

Your mongodb database lives inside the ~/apps/nightscout/data folder as specified in the docker-compose.yml file on line 34
```
    volumes:
      - ./data:/data/db
```
You can download this folder with an SFTP connection to your VPS and back it up before reinstalling your vps or migrating to another VPS. By recreating the exact same folder structure and running the docker-compose.yml from the same root directory you will be able to continue using the same database.



