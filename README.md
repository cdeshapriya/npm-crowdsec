
# Nginx Proxy Manager Configuration with Crowdsec

## What you will need
- A small amount of experience using docker
- A machine with docker already set up for this guide I will be using a portainer instance
- A domain already pointing at your old Nginx proxy manager instance (I assume that you set up with a database)

## Setting Up Nginx Proxy Manager

First, we will need to set up Nginx Proxy Manager via its docker-compose I will be using baudneo fork because we will need to use that fork later anyway. Make sure that you set your database passwords to something else.

you will also need to run the following before you run `docker-compose up -d`

```
mkdir /Nginx-Proxy-Manger
```
```
touch /Nginx-Proxy-Manger/crowdsec-openresty-bouncer.conf
```
