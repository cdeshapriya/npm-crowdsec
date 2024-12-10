# Introduction

Nginx Proxy Manager (NPM) is a popular Docker-based, user-friendly webproxy-in-a-box solution. Supporting CrowdSec for NPM is fantastic because it offers an additional layer of security to the websites you are proxying. CrowdSec with Nginx Proxy Manager provides comprehensive security for your web applications.

But what about native support? That would be awesome, right? Fortunately, someone had that thinking on May 28, 2021, when they raised an issue about it on the project's GitHub repository.

CrowdSec was not as popular back then as it is now, thus nothing much happened for the first few months following that save for GitHub users adding comments to support the notion of the issue's original developer.

Someone tried adding CrowdSec in October 2021, but there was no suitable log parser at the time because the NPM project changed the logfile format, making it impossible to parse with the standard NGINX log parser.

All of that changed in January 2022, when the CrowdSec developer team finally decided to create one. And when a component for OpenResty (the Nginx-based web platform used by NPM) became available, the fun could begin!.






# Nginx Proxy Manager Configuration with Crowdsec

## What you will need
- A small amount of experience using docker
- A machine with docker already set up for this guide I will be using a portainer instance
- A domain already pointing at your old Nginx proxy manager instance (I assume that you set up with a database)

## Setting Up Nginx Proxy Manager

First, we will need to set up Nginx Proxy Manager via its docker-compose I will be using baudneo fork because we will need to use that fork later anyway. Make sure that you set your database passwords to something else.

you will also need to run the following before you run `docker-compose up -d`

```
mkdir /opt/nginx-proxy-manager
```

```
touch /opt/nginx-proxy-manger/crowdsec-openresty-bouncer.conf
```

```
# https://hub.docker.com/r/baudneo/nginx-proxy-manager
# https://www.crowdsec.net/blog/crowdsec-with-nginx-proxy-manager
version: "3"
services:
  app:
    # image: 'jc21/nginx-proxy-manager:latest'
    image: 'baudneo/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format :
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    environment:
      # crowdsec config
      TZ: Australia/Perth
      ADMIN_PANEL_LOG: "1"
      CROWDSEC_BOUNCER: "1"
      OPENRESTY_DEBUG: "0"

      CROWDSEC_LAPI: "http://crowdsecip:8080/"
      CROWDSEC_KEY: "yourkeyhere"
      
    
      DB_MYSQL_HOST: "172.25.0.3"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "Verygoodpassword"
      DB_MYSQL_NAME: "npm"
      # Uncomment this if IPv6 is not enabled on your host
      # DISABLE_IPV6: 'true'
    volumes:
      - /Nginx-Proxy-Manger/data:/data
      - /Nginx-Proxy-Manger/letsencrypt:/etc/letsencrypt
      - /Nginx-Proxy-Manger/crowdsec-openresty-bouncer.conf:/data/crowdsec/crowdsec-openresty-bouncer.conf
    depends_on:
      - db
    networks:
      nginx:
        ipv4_address: 172.25.0.2

  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'Verygoodpassword'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'Verygoodpassword'
    volumes:
      - /Nginx-Proxy-Manger/data/mysql:/var/lib/mysql
    networks:
      nginx:
        ipv4_address: 172.25.0.3

networks:
  nginx:
    ipam:
      driver: default
      config:
        - subnet: 172.25.0.0/16
```

```
```
