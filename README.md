# funkwhale_tailscale

This is modified docker compose to help deploy funkwhale in a private tailscale group.

I want to share music with my devices and some friends and I have had bad experiences self-hosting open source projects, in that if I don't regularly watch them and update dependencies, they often end up hacked. This should be fine for a "fire and forget" deploy for small groups, but will not work with federation.

Much thanks to https://github.com/sage2050/funkwhale_unraid which I forked for this project.

## Prerequistes

You should have set up a server (Hetzner, Linode, DigitalOcean, etc). Please install docker and tailscale, then login with tailscale to your group.

Setup your hostname as `funkwhale`, which will also be used as the name to find it on tailscale.

## Setup

First, ssh to your new server.

1. Open terminal and run the following command. you will use the output as your Django secret key
   - openssl rand -base64 45
2. open the .env file in a text editor and configure the following variables
   - FUNKWHALE_API_IP
   - FUNKWHALE_API_PORT
   - FUNKWHALE_HOSTNAME
   - EMAIL_CONFIG (if desired)
   - ACCOUNT_EMAIL_VERIFICATION (if desired)
   - MEDIA_ROOT (music uploads from external users will go here)
   - DJANGO_SECRET_KEY (use the rand from step 1)
   - MUSIC_DIRECTORY_SERVE_PATH
3. open funkwhale.subdomain.conf and edit $upstream_app to point to your unraid server IP (NOT the name of the container)
4. create a funkwhale appdata folder and move docker-compose.yml and .env to it (/mnt/user/appdata/funkwhale)
5. In unraid terminal, navigate to the appdata folder (cd /mnt/user/appdata/funkwhale)
6. run "docker compose up -d postgres"
7. run "docker compose run --rm api funkwhale-manage migrate"
8. run "docker compose run --rm api funkwhale-manage fw users create --superuser" to create an admin account
9. run "docker compose up -d"
10. move funkwhale.subdomain.conf to your SWAG proxyconfs folder (/mnt/user/appdata/swag/nginx/proxyconfs)
11. restart the SWAG container

set up your cname on cloudflare/duckdns/etc and you should be able to navigate to your funkwhale instance and login with your superuser account (you will NOT be able to login if you navigate to [IP]:[PORT] )

note: the volume mappings will look incorrect but leave them as is. the docker-compose file does not need to be changed
