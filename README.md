# self-hosted-services

This repository contains everything you need to start self-hosting a core set of privacy-preserving services that I have found helpful, all run via a common [Docker Compose](https://docs.docker.com/compose/) configuration using Let's Encrypt for SSL certificates.

# Requirements

* [Git](https://git-scm.com/downloads)
* [Docker Engine](https://docs.docker.com/engine/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* Ports `80/tcp`, `443/tcp`, `18080/tcp`, and `18089/tcp` exposed/forwarded to the host
* DNS entries for your domain and each desired sub-domain

# Included Services

* [Nextcloud](https://github.com/nextcloud/server)
  * A self-hosted server for hosting files, photos, backups, contacts, calendars, and much more
* [Monero](https://github.com/sethsimmons/simple-monerod-docker)
  * A self-sovereign, private-by-default, digital cash
  * This service runs a pruned Monero node, allowing you to connect your own Monero wallet(s) directly to your own node
* [Nitter](https://github.com/zedeus/nitter)
  * A privacy-preserving Twitter front-end
* [Teddit](https://codeberg.org/teddit/teddit)
  * A privacy-preserving Reddit front-end
* [Wallabag](https://github.com/wallabag/wallabag)
  * A privacy-preserving article reader in the vein of Pocket
* [Yotter](https://github.com/ytorg/yotter)
  * A privacy-preserving YouTube and Twitter front-end

# How does it work?

This repo relies on Docker Compose to configure and run all of the above services, leveraging [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) and [docker-letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion) to automatically expose each service, request and maintain Let's Encrypt certificates for SSL, and handle all proxying.

# Getting Started

You will need to clone this repository to the host you want running these services first:

`git clone https://github.com/sethsimmons/self-hosted-services.git`
`cd self-hosted-services`

Once cloned, set the necessary passwords and desired sub-domains in the `.env` file (*PLEASE DO NOT COPY THE FOLLOWING CONFIG, CHANGE THE PASSWORDS AND HOSTNAMES APPROPRIATELY IN YOUR LOCAL `.env` FILE*):

```
# Yotter Variables
YOTTER_HOSTNAME=yotter.mydomain.com
YOTTER_DB_ROOT_PASS=changeme
YOTTER_DB_PASS=changeme

# Nextcloud Variables
NEXTCLOUD_HOSTNAME=nextcloud.mydomain.com
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=changeme
POSTGRES_PASSWORD=changeme

# Wallabag Variables
WALLABAG_HOSTNAME=wallabag.mydomain.com
WALLABAG_DB_ROOT_PASS=changeme
WALLABAG_DB_PASS=changeme
WALLABAG_URL=https://wallabag.mydomain.com

# Teddit Variables
TEDDIT_HOSTNAME=teddit.mydomain.com

# Nitter Variables
NITTER_HOSTNAME=nitter.mydomain.com

# Let's Encrypt Variables
LE_EMAIL_ADDRESS=myemail@pm.me
```

Note that all hostnames used must already have DNS entries configured with your domain provider in order for certificate generation to function properly.

Edit the Nitter, Teddit, and Yotter configuration files:

* Nitter
  * Replace the `hostname`, `replaceTwitter`, and `replaceYouTube` values with the relevant hostnames
* Teddit
  * Replace the `domain` value with the proper Teddit hostname
* Yotter
  * Replace the `serverName` and `nitterInstance` values with the relevant hostnames

Start-up the services with Docker Compose:

`docker-compose up -d`

# Auto-healing and Updates

Each service included has a rudimentary health-check configured to allow [docker-autoheal](https://github.com/willfarrell/docker-autoheal) to restart any services that may fail.

Automatic updates are provided by the [Watchtower](https://containrrr.dev/watchtower/) container that watches and updates base images of services when available.