# nightscout-docker-simple
Simple Nightscout CGM setup in docker on a VPS

This is a simple nightscout docker setup that I use on my VPS running Ubuntu 22.04 in combination with [NginxProxyManager/nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager).

It is inspired by [LostOnTheLine/Nightscout_Docker-Compose](https://github.com/LostOnTheLine/Nightscout_Docker-Compose) and [dhermanns/rpi-nightscout](https://github.com/dhermanns/rpi-nightscout).

I'm running the nightscout docker container as root alongside other containers without problems.

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
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
3. Verify that the Docker Engine installation is successful by running the hello-world image.
```
sudo docker run hello-world
```

### Step 3 - Install and configure Nginx Proxy Manager

Create a folder “apps” to hold all of the docker-compose scripts, and create a folder for nginxproxymanager inside that folder.  

```
mkdir apps
cd apps
mkdir nginxproxymanager
cd nginxproxymanager
```

Create a docker-compose file

```
nano docker-compose.yml
```

Copy the following code into the file.

```
version: "3"
services:
  app:
  image: 'jc21/nginx-proxy-manager:latest'
  restart: unless-stopped
  ports:
  # These ports are in format <host-port>:<container-port>
  - '80:80' # Public HTTP Port
  - '443:443' # Public HTTPS Port
  - '81:81' # Admin Web Port
  # Add any other Stream port you want to expose
  # - '21:21' # FTP

  # Uncomment the next line if you uncomment anything in the section
  # environment:
  # Uncomment this if you want to change the location of
  # the SQLite DB file within the container
  # DB_SQLITE_FILE: "/data/database.sqlite"

  # Uncomment this if IPv6 is not enabled on your host
  # DISABLE_IPV6: 'true'

  volumes:
  - ./data:/data
  - ./letsencrypt:/etc/letsencrypt
```

Save with ctrl + x and run `docker compose up -d` to start the container. Go to a web browser and type in the address of your server like so: `127.0.0.0:81`.

Log in using the default credentials admin@example.com and password “changeme” and it will prompt you to create new credentials. We will come back here later.

### Step 4 - Install and configure nightscout

Navigate back to ~/apps and create a new directory for your nightscout app, this is where your nightscout will live, including the mongodb

```
mkdir nightscout
cd nightscout
nano docker-compose.yml
```
Copy the following code into the file. Make sure you change the API_SECRET key to something at least 12 characters long. You can edit the plugins here too (careportal, iob etc..) For a comprehensive overview see the [nightscout](https://nightscout.github.io/) documentation.

```
version: '3.9'
services:
  nightscout:
    image: nightscout/cgm-remote-monitor:latest
    environment:
      TZ: Europe/Amsterdam
      MONGO_CONNECTION: mongodb://mongo:27017/nightscout
      API_SECRET: secretAPIatLeast12Characters
      CUSTOM_TITLE: "Nightscout"
      TIME_FORMAT: 24
      THEME: colors 
      BG_HIGH: 220
      BG_LOW: 60
      BG_TARGET_TOP: 180
      BG_TARGET_BOTTOM: 80
      DISPLAY_UNITS: mmol/L
      BASAL_RENDER: default  
      INSECURE_USE_HTTP: "true"
      AUTH_DEFAULT_ROLES: readable devicestatus-upload
      ENABLE: careportal food iob cob basal bage iage sage cage bolus pump openaps override cors boluscalc bwp
      SHOW_PLUGINS: pump rawbg iob cob
      EDIT_MODE: off 
      ALARM_URGENT_HIGH: off
      ALARM_HIGH: off
      ALARM_LOW: off
      ALARM_URGENT_LOW: off
    ports:
      - "1337:1337"
    depends_on:
      - mongo
    restart: always
  mongo:
    image: mongo:4.4.9
    volumes:
      - ./data:/data/db                                                                      
    ports:
      - "27017:27017"
      - "27018:27018"
      - "27019:27019"
      - "28017:28017"
    restart: always
```
Save with ctrl + x and run `docker compose up -d` to start the container.

If everything went well you can reach your nightscout now in the browser with the direct IP of your VPS on port 1337. e.g. 127.0.0.0:1337

### Step 5 - Configuring the proxy and SSL certificates with Nginx Proxy Manager

Head over to where you registered your domain to set up an A record pointed at your server IP. You’ll set the host to “@” and the value to your instances IP address. If you plan on having multiple domains going to multiple instances, they will all point at the same IP — the proxy manager we installed will take care of directing traffic to the appropriate container.

Head back into nginx proxy manager, login and head to hosts < proxy hosts < add proxy host. Add your domain name, the ip of your server and the port we set up earlier (1337). 

Then go to the SSL tab, and request a new SSL certificate. Enable "Force SSL."

Done. You can now reach your nightscout site from your domain over https.

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
docker compose up -d
```

Alternatively you can make changes to your docker-compose.yml file while its still running and restart it with the following command. But make sure to add the --remove-orphans argument in this case.
```
docker compose up -d --remove-orphans
```

### Migrate or reinstall VPS

Your mongodb database lives inside the ~/apps/nightscout/data folder as specified in the docker-compose.yml file on line 34
```
    volumes:
      - ./data:/data/db
```
You can download this folder with an SFTP connection to your VPS and back it up before reinstalling your vps or migrating to another VPS. By recreating the exact same folder structure and running the docker-compose.yml from the same root directory you will be able to continue using the same database, without having androidAPS or xDrip+ having to sync again. It is good practice to make a backup of your database from time to time.



