# self-hosted-services

This repository contains everything you need to start self-hosting a core set of privacy-preserving services that I have found helpful, all run via a common [Docker Compose](https://docs.docker.com/compose/) configuration using [Let's Encrypt](https://letsencrypt.org/) for SSL certificates.

***NOTE: I am in the process of migrating all apps here to [meienberger/runtipi](https://github.com/meienberger/runtipi), as that is a much better overall solution for this type of project. Once done I will archive this repo and direct all users to that. You can follow the porting process here: https://github.com/meienberger/runtipi-appstore/pulls/sethforprivacy***

## Requirements

* [Git](https://git-scm.com/downloads)
* [Docker Engine](https://docs.docker.com/engine/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* Ports `80/tcp`, `443/tcp`, `18080/tcp`, `18083/tcp`, `3333/tcp`, `37889/tcp`, and `18089/tcp` exposed/forwarded to the host
* DNS entries for your top-level domain and each desired sub-domain

## Included Services

* [Nextcloud](https://github.com/nextcloud/server)
  * A self-hosted server for hosting files, photos, backups, contacts, calendars, and much more
* [Uptime-kuma](https://github.com/louislam/uptime-kuma)
  * It is a self-hosted monitoring tool like "Uptime Robot"
* [Libretranslate](https://github.com/LibreTranslate/LibreTranslate)
  * Free and Open Source Machine Translation API, entirely self-hosted
* [SearXNG](https://github.com/searxng/searxng)
  * a privacy-respecting, hackable metasearch engine
* [Send](https://github.com/timvisee/send)
  * a fork of Mozillas's Firefox Send, which was an encrypted file sharing tool
* [Wordpress](https://github.com/WordPress/WordPress)
  * WordPress is a free and open-source content management system (Website Builder)
* [Cryptpad](https://github.com/xwiki-labs/cryptpad)
  * CryptPad is a collaboration suite that is end-to-end-encrypted and open-source
* [P2Pool](https://github.com/SChernykh/p2pool)
  * Decentralized pool for Monero mining
* [Invidious](https://github.com/iv-org/invidious)
  * Invidious is an alternative front-end to YouTube 
* [Monero](https://github.com/sethforprivacy/simple-monerod-docker)
  * A self-sovereign, private-by-default, digital cash
  * This service runs a pruned Monero node, allowing you to connect your own Monero wallet(s) directly to your own node
* [Nitter](https://github.com/zedeus/nitter)
  * A privacy-preserving Twitter front-end
* [PrivateBin](https://privatebin.info/)
  * A privacy-preserving and encrypted-by-default pastebin
* [Teddit](https://codeberg.org/teddit/teddit)
  * A privacy-preserving Reddit front-end
* [Wallabag](https://github.com/wallabag/wallabag)
  * A privacy-preserving article reader in the vein of Pocket
* [Monero blockchain explorer](https://github.com/moneroexamples/onion-monero-blockchain-explorer)
  * A self-hosted Monero blockchain explorer
* [Conduit](https://conduit.rs)
  * A self-hosted Matrix homeserver written in Rust
* [Jellyfin](https://github.com/jellyfin/jellyfin)
  * A software media system with no strings attached, no premium licenses or features, and no hidden agendas

*NOTE: If you do not want to run one of the services above simply comment out or delete the relevant service section from `docker-compose.yml`.*

## How does it work?

This repo relies on Docker Compose to configure and run all of the above services, leveraging Traefik to automatically expose each service, [request and maintain](https://doc.traefik.io/traefik/user-guides/docker-compose/acme-tls/) Let's Encrypt certificates for SSL, and handle all proxying.

## Starting the Services

You will need to clone this repository to the host you want running these services first:

```bash
git clone https://github.com/sethforprivacy/self-hosted-services.git
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

# Privatebin Variables
PRIVATEBIN_HOSTNAME=paste.mydomain.com

# Monero explorer variables
EXPLORER_HOSTNAME=explorer.mydomain.com

# Let's Encrypt Variables
LE_EMAIL_ADDRESS=myemail@pm.me
```

Note that all hostnames used must already have DNS entries configured with your domain provider in order for certificate generation to function properly.

Edit the Nitter configuration file:

* Nitter
  * Replace the `hostname`, `replaceTwitter`, and `replaceYouTube` values with the relevant hostnames

* SearXNG
  * Generate MORTY_KEY `sed -i "s|ReplaceWithARealKey\!|$(openssl rand -base64 33)|g" settings.yml .env`
  * go into the settings.yml file and replace the searx.mydomain.com at the bottom of the file with your domain.

* P2Pool
  * Hugepages are very important for optimal mining performance. You need to enable them first using the following commands:
``` bash
sudo sysctl vm.nr_hugepages=3072
sudo bash -c "echo vm.nr_hugepages=3072 >> /etc/sysctl.conf"
```
Note: If your VPS or Server does not have enough RAM you can reduce it by using 1168 instead of 3072

* Conduit
  * Start by planning out your setup. If you are willing to dedicate the domain you want to use as the handle:
  * @user:domain.tld, then just set that in Conduit_Hostname in .env and in conduit/wellknown.conf as the whole <subdomain>.<domain> part
  * If you wish to have a website, or other stuff on that domain instead, then you will need to dive-in some way or another.
  * Let's say you're okay with having the handle look like @user:subdomain.domain.tld, then just set that in the files mentioned above.
  * If you wish, to have the domain in the handle, and in use at the same time, you will need to dive into the docker-compose.yml itself
  * and set the line with comment: `#Change to root domain if desired` to the root domain, otherwise keep the subdomain everywhere else
  * Set up redirect on the route `https://domain.tld/.well-known/matrix/*` to `https://subdomain.domain.tld/.well-known/matrix/:splat`
  * At least this setup was tested. Feel free to come up with easier ways to this setup.
  * Otherwise if you get stuck, leave an issue, someone will surely help out.

Start-up the services with Docker Compose:

* `docker-compose up -d`

## Tor Support

Currently Monero, P2Pool, Nitter, SearXNG, Invidious, Libretranslate, and Teddit get default Tor support, but I will likely expand that in the future. To list Onion services, simply run:

* `docker exec -ti tor onions`

## Updates

Automatic updates are provided by the [Watchtower](https://containrrr.dev/watchtower/) container that watches and updates base images of services when available. It will automatically search for, download, and migrate your services to updated images whenever available.

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
* [PrivateBin](https://privatebin.info/)
* [Teddit](https://codeberg.org/teddit/teddit)
* [Wallabag](https://www.wallabag.it/en/features)
* [Invidious](https://docs.invidious.io/)
* [P2Pool](https://github.com/SChernykh/p2pool)
* [CryptPad](https://docs.cryptpad.fr/en/index.html)
* [WordPress](https://wordpress.org/support/)
* [Send](https://github.com/timvisee/send/tree/master/docs)
* [Uptime-kuma](https://github.com/louislam/uptime-kuma/wiki/)
* [SearXNG](https://searxng.github.io/searxng/)
* [Libretranslate](https://github.com/LibreTranslate/LibreTranslate)
* [Conduit](https://gitlab.com/famedly/conduit#how-can-i-deploy-my-own)
* [Jellyfin](https://jellyfin.org/docs)

## Donations

If you decide to run this and use these services, please don't forget to donate to those people making these services a reality!

* [Monero](https://ccs.getmonero.org/funding-required/)
* [Nitter](https://github.com/zedeus/nitter#nitter)
* [Teddit](https://codeberg.org/teddit/teddit#teddit)
* [Wallabag](https://liberapay.com/wallabag/donate)
* [Invidious](https://github.com/iv-org/invidious#donate)
* [P2Pool](https://github.com/SChernykh/p2pool#donations)
* [CryptPad](https://github.com/xwiki-labs/cryptpad)
* [WordPress](https://wordpressfoundation.org/donate/)
* [Uptime-kuma](https://opencollective.com/uptime-kuma)
* [SearXNG](https://www.searx.me/static/donate.html)
* [Conduit](https://conduit.rs/#donate)
* [Jellyfin](https://opencollective.com/jellyfin)

## Additional Resources

* [Docker Compose documentation](https://docs.docker.com/compose/)

## Additional Credits

* https://github.com/cmehay/docker-tor-hidden-service
  * This Docker container makes it incredibly easy to expose Tor Hidden Services of other running containers
* https://github.com/containrrr/watchtower
  * Watchtower automates updating base images for other running containers
