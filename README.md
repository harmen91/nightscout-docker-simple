# nightscout-docker-simple
Simple Nightscout CGM setup in docker on a VPS

This is a simple nightscout docker setup that I use on my VPS running Ubuntu 22.04 in combination with nginx proxy manager (NginxProxyManager/nginx-proxy-manager).

It is inspired by [LostOnTheLine/Nightscout_Docker-Compose](https://github.com/LostOnTheLine/Nightscout_Docker-Compose) to strip it from the traefik ssl encription and [dhermanns/rpi-nightscout](https://github.com/dhermanns/rpi-nightscout) for its simplicity.

I'm running this docker container as root alongside other containers without problems.

# Guide

Install and set up docker and nginx-proxy-manager as per [this guide](https://medium.com/@jmpinney/multiple-wordpress-sites-on-one-server-with-docker-6fb53adc4bfe) , ignore the wordpress bit 

Navigate back to ~/apps and create a new directory for your nightscout app, this is where your nightscout will live, including the mongodb

`mkdir nightscout
`
`cd nightscout
`
`nano docker-compose.yml
`
copy paste the content of docker-compose.yml from this repository. Make sure to change the API Secret key to something at least 12 characters long. You can edit the plugins here too (careportal, iob etc..) For a comprehensive overview see the [nightscout](https://nightscout.github.io/) documentation.

`docker compose up -d --remove-orphans
`
You can reach your nightscout now in the browser on the direct IP of your VPS on port 1337. e.g. 182.229.239.135:1337

Continue with the last bit of [this guide](https://medium.com/@jmpinney/multiple-wordpress-sites-on-one-server-with-docker-6fb53adc4bfe) to generate SSL certificates for your site


# Edit configuration / stop /start & migrate

To stop, edit and start your nightscout configuration again run
`docker ps
`
note the CONTAINER ID of nightscout and monogdb

`docker stop CONTAINER ID
`

Your database entries live inside the ~/apps/nightscout/data folder
You can download this folder with an SFTP connection to your VPS and back it up before reinstalling your vps or migrating to another VPS



