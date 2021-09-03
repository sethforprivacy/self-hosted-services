# self-hosted-services

This repository contains everything you need to start self-hosting a core set of privacy-preserving services that I have found helpful, all run via a common [Docker Compose](https://docs.docker.com/compose/) configuration using [Let's Encrypt](https://letsencrypt.org/) for SSL certificates.

## Requirements

* [Git](https://git-scm.com/downloads)
* [Docker Engine](https://docs.docker.com/engine/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* Ports `443/tcp`, `18080/tcp`, and `18089/tcp` exposed/forwarded to the host
* DNS entries for your top-level domain and each desired sub-domain

## Included Services

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
* [Heimdall Dashboard](https://heimdall.site/)
  * A dashboard for viewing stats and status of services running behind Traefik

*NOTE: If you do not want to run one of the services above simply comment out or delete the relevant service section from `docker-compose.yml`.*

## How does it work?

This repo relies on Docker Compose to configure and run all of the above services, leveraging Traefik to automatically expose each service, [request and maintain](https://doc.traefik.io/traefik/user-guides/docker-compose/acme-tls/) Let's Encrypt certificates for SSL, and handle all proxying.

## Starting the Services

You will need to clone this repository to the host you want running these services first:

```bash
git clone https://github.com/sethsimmons/self-hosted-services.git
cd self-hosted-services
```

Once cloned, set the necessary passwords and desired sub-domains in the `.env` file (*PLEASE DO NOT COPY THE FOLLOWING CONFIG, CHANGE THE PASSWORDS AND HOSTNAMES APPROPRIATELY IN YOUR LOCAL `.env` FILE*):

```ini
# Nextcloud Variables
NEXTCLOUD_HOSTNAME=nextcloud.mydomain.com
NEXTCLOUD_ADMIN_USER=
NEXTCLOUD_ADMIN_PASSWORD=
POSTGRES_PASSWORD=

# Wallabag Variables
WALLABAG_HOSTNAME=wallabag.mydomain.com
WALLABAG_DB_ROOT_PASS=
WALLABAG_DB_PASS=
WALLABAG_URL=https://wallabag.mydomain.com

# Teddit Variables
TEDDIT_HOSTNAME=teddit.mydomain.com

# Nitter Variables
NITTER_HOSTNAME=nitter.mydomain.com

# Traefik Variables
TRAEFIK_HOSTNAME=traefik.mydomain.com

# Heimdall Variables
DASHBOARD_HOSTNAME=dashboard.mydomain.com
DASHBOARD_TZ=America/New York

# Let's Encrypt Variables
LE_EMAIL_ADDRESS=myemail@pm.me
```

Note that all hostnames used must already have DNS entries configured with your domain provider in order for certificate generation to function properly.

Edit the Nitter configuration file:

* Nitter
  * Replace the `hostname`, `replaceTwitter`, and `replaceYouTube` values with the relevant hostnames

Start-up the services with Docker Compose:

* `docker-compose up -d`

## Tor Support

Currently only Monero, Nitter, and Teddit get default Tor support, but I will likely expand that in the future. To list Onion services, simply run:

* `docker exec -ti tor onions`

## Auto-healing and Updates

Each service included has a rudimentary health-check configured to allow [docker-autoheal](https://github.com/willfarrell/docker-autoheal) to restart any services that may fail.

Automatic updates are provided by the [Watchtower](https://containrrr.dev/watchtower/) container that watches and updates base images of services when available.

## Logging

If you find yourself in need of viewing logs for a given service, simply run the following to tail all logs:

```bash
docker-compose logs --follow
```

To view the logs of a single service, run:

```bash
docker-compose logs --follow <service_name>
```

i.e.:

```bash
docker-compose logs --follow monerod
```

## Getting Started

As this simply helps you get these services running, using each service is outside of the scope of this project. However, below are some links for getting started with each:

* [Nextcloud](https://docs.nextcloud.com/server/21/user_manual/en/)
* [Monero](https://sethforprivacy.com/guides/run-a-monero-node/#sending-commands-to-your-node)
* [Nitter](https://nitter.net/about)
* [Teddit](https://codeberg.org/teddit/teddit)
* [Wallabag](https://www.wallabag.it/en/features)

## Donations

If you decide to run this and use these services, please don't forget to donate to those people making these services a reality!

* [Monero](https://ccs.getmonero.org/funding-required/)
* [Nitter](https://github.com/zedeus/nitter#nitter)
* [Teddit](https://codeberg.org/teddit/teddit#teddit)
* [Wallabag](https://liberapay.com/wallabag/donate)
* [Heimdall](https://github.com/linuxserver/Heimdall)

## Potential Future Services

* [Piped](https://github.com/TeamPiped/Piped)
  * A privacy-preserving YouTube front-end
* [Plex](https://docs.linuxserver.io/images/docker-plex)
  * A self-hosted media server

## Additional Resources

* [Docker Compose documentation](https://docs.docker.com/compose/)

## Additional Credits

* https://github.com/cmehay/docker-tor-hidden-service
  * This Docker container makes it incredibly easy to expose Tor Hidden Services of other running containers
* https://github.com/containrrr/watchtower
  * Watchtower automates updating base images for other running containers
* https://github.com/willfarrell/docker-autoheal
  * Utilizes Docker-Compose healthchecks to restart containers that fail their healthcheck for any reason
