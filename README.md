# funkwhale_tailscale

This is modified docker compose to help deploy funkwhale in a private tailscale group.

I want to share music with my devices and some friends and I have had bad experiences self-hosting open source projects, in that if I don't regularly watch them and update dependencies, they often end up hacked. This should be fine for a "fire and forget" deploy for small groups, but will not work with federation.

Much thanks to https://github.com/sage2050/funkwhale_unraid which I forked for this project.

## Prerequistes

You should have set up a server (Hetzner, Linode, DigitalOcean, etc). Please install docker and tailscale, then login with tailscale to your group.

Setup your hostname as `funkwhale`, which will also be used as the name to find it on tailscale.

## Setup

### Music Directories

First, ssh to your new server. And set up two folders.

The first one will hold music you upload, you can use a command like `scp -r /my/local/music/dir root@funkwhale:/root/music` to upload your music. You will refer to this directory later as `MUSIC_DIRECTORY_SERVE_PATH`.

The second one should be empty but on a volume with significant space to grow. You will refer to this directory later as `MEDIA_ROOT`.

### Checkout Repo

Now, go to your home directory (`/root` ?) checkout the repo:

```bash
git clone https://github.com/pomofomo/funkwhale_tailscale.git funkwhale
cd funkwhale
```

### Env File

1. Open terminal and run the following command. you will use the output as your Django secret key
   - `openssl rand -base64 45`
2. open the .env file in a text editor and configure the following variables
   - `DJANGO_SECRET_KEY` (use the rand from step 1)
   - `FUNKWHALE_HOSTNAME` (your complete tailscale hostname, e.g. `funkwhale.tail123456.ts.net`)
   - `MEDIA_ROOT` (music uploads from external users will go here)
   - `MUSIC_DIRECTORY_SERVE_PATH`
   - `EMAIL_CONFIG` (if desired)
   - `ACCOUNT_EMAIL_VERIFICATION` (if desired)

### Start with docker

1. run `docker compose up -d postgres`
2. run `docker compose run --rm api funkwhale-manage migrate`
3. run `docker compose run --rm api funkwhale-manage fw users create --superuser` to create an admin account
4. run `docker compose up -d` to start all the processes
5. run `docker ps` to view all the processes are running 

### Tailscale Proxy

This should now be available at http://funkwhale.tail123456.ts.net, but we want this to work on https.

Follow the instructions here to enable http certs: https://tailscale.com/kb/1153/enabling-https 

```bash
tailscale cert
tailscale serve --bg 80
```

## Connect from a client

You should now be able to connect from any other device on the same tailscale network.

Just go to http://funkwhale.tail123456.ts.net or whatever your real tailscale name is

## Let to do

Ensure it **only** binds to the tailscale network. See https://tailscale.com/kb/1282/docker
