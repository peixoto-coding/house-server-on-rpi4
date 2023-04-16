# House Server on a Raspberry Pi 4 with Docker

## About

This project aims to self-host every service I can in my own house, under my own rules, with minimal to no exposition to third party providers.
Using a Raspberry Pi or another computer, you can do this yourself. Using Docker makes it easy to configure and to migrate to another device, when needed.

I started with Home Assistant, for house devices control, Passbolt password management for my household and Traefik to route incoming traffic to the correct services.
One can also use DuckDNS and Let's Encrypt to get SSL certificates and improve security.

## Requirements

To get this to run, the following are needed.
* A computer
* Docker
* DuckDNS account
* Portforward in your router to your computer

You can follow the DuckDNS installation [here](https://www.duckdns.org/install.jsp).


## Instalation

1. Clone the repo
   ```git clone https://github.com/peixoto-coding/house-server-on-rpi4.git```
2. Create a folder for your Home Assistant configuration files, for example:
   ```mkdir home-assistant-config```
3. If you don't have a media folder in the device, create one for Home Assistant, for example:
   ```mkdir home-assistant-media```
4. Change the docker-compose.yaml file to reflect your folders, your DuckDNS domain and set a password for your Passbolt database
   ```services:
        homeassistant:
          ...
          labels:
            ...
            traefik.http.routers.homeassistant-https.rule: "Host(`homeassistant.<your-domain>.duckdns.org`)"
            ...
          volumes:
            - <your-path-to-HA-configuration-folder>:/config
            - /etc/localtime:/etc/localtime
            - <your-path-to-media-folder>:/media
        ...
        passbolt-db:
          ...
          environment:
            ...
            MYSQL_PASSWORD: "<your-secure-password>"
        ...
        passbolt:
          ...
          environment:
            APP_FULL_BASE_URL: https://passbolt.<your-domain>.duckdns.org/
            DATASOURCES_DEFAULT_HOST: "passbolt-db"
            DATASOURCES_DEFAULT_USERNAME: "passbolt"
            DATASOURCES_DEFAULT_PASSWORD: "<your-secure-password>"
            DATASOURCES_DEFAULT_DATABASE: "passbolt"
          ...
          labels:
            ...
            traefik.http.routers.passbolt-http.entrypoints: "web"
            traefik.http.routers.passbolt-http.rule: "Host(`passbolt.<your-domain>.duckdns.org`)"
            ...
            traefik.http.routers.passbolt-https.rule: "Host(`passbolt.<your-domain>.duckdns.org`)"
            ...
```
```
5. Change your email in traefik.yaml file:
```certificatesResolvers:
  letsencrypt:
    acme:
      email: <your-email-here>
```
6. Run the docker-compose.yaml file:
```docker-compose -f docker-compose.yaml up -d```
7. Create Passbolt superuser, change the email, first and last name before running the command:
```docker-compose -f docker-compose.yaml exec passbolt su -m -c "/usr/share/php/passbolt/bin/cake \
                                passbolt register_user \
                                -u <your-email> \
                                -f <first-name> \
                                -l <last-name> \
                                -r admin" -s /bin/sh www-data
```
8. If the command is correctly executed, Passbolt should provide an URL to do the first setup.
9. You should now be able to access your services on the internet by going to the urls you defined, for example the `https://homeassistant.<your-domain>.duckcns.org`


# Coming in the future

The idea is to expand this project and my Raspberry Pi with more services that are useful to the household, but keeping everything always self-hosted.
