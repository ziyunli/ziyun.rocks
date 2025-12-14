---
title: "Self-hosting ArchiveBox on a VPS"
slug: self-hosting-archivebox-on-a-vps
date: 2021-04-24T22:16:31-04:00
summary: "A step-by-step guide to self-hosting ArchiveBox on a Vultr One-Click Docker instance. Covers Docker Compose setup, firewall configuration with ufw, and publishing via Caddy reverse proxy with automatic SSL."
---

Note: this article is edited and published at [Vultr Docs](https://www.vultr.com/docs/install-archivebox-on-a-oneclick-docker-application)

[ArchiveBox](https://archivebox.io) is a powerful, self-hosted internet archiving solution to collect, save, and view sites you want to preserve offline.
This guide explains how to self-host ArchiveBox on a Vultr One-Click Docker application, and publish it with a Caddy reverse proxy.

<!--more-->

# Prerequisites

* A Vultr One-Click Docker application running Ubuntu 18.04
* Caddy

This guide assumes that only ArchiveBox is hosted on the server, but you can easily extend the configuration of Caddy for more applications.

# 1. Set up ArchiveBox

Using `docker-compose` is the recommended way to set up ArchiveBox. And ArchiveBox provides an official `docker-compose.yml` which bundles all dependencies that we can use to set up our server.

1. After One-Click Docker deploys, [log in as root via SSH](https://www.vultr.com/docs/how-to-access-your-vultr-vps).

2. Switch to the `docker` user

```bash
su - docker
```
3. Create a new empty directory, and download the official [`docker-compose.yml`](https://raw.githubusercontent.com/ArchiveBox/ArchiveBox/master/docker-compose.yml) file. Note this folder will also be the place to store data of ArchiveBox.

```bash
mkdir ~/archivebox && cd ~/archivebox
curl -O 'https://raw.githubusercontent.com/ArchiveBox/ArchiveBox/master/docker-compose.yml'
```

4. (Optional) You can set a [restart policy](https://docs.docker.com/config/containers/start-containers-automatically/#use-a-restart-policy) of the `archivebox` service in the downloaded `docker-compose.yml`, so that ArchiveBox can start automatically on different situations. For example, set it to `always`.

```yaml
archivebox:
    image: ${DOCKER_IMAGE:-archivebox/archivebox:latest}
        command: server --quick-init 0.0.0.0:8000
    ports:
        - 8000:8000
    restart: always
    environment:
        - ALLOWED_HOSTS=*
        - MEDIA_MAX_SIZE=750m
    volumes:
        - ./data:/data.
```

5. Run the initial setup and create an admin user. You will use this admin user to create bookmarks in ArchiveBox.

```bash
docker-compose run archivebox init --setup
```

6. Start the server at `localhost:8000`

```bash
docker-compose up -d
```

7. Switch back to the root user by pressing `Ctrl+D`

# 2. Set up a server firewall

A firewall prevents access to our server via un-allowed ports. For ArchiveBox, we only need to expose port 80 (for HTTP) and 443 (for HTTPS). You can also enable the SSH port which is helpful for a lot of situations, but it's optional. In this guide, we will use the Uncomplicated Firewall `ufw` that is a front-end for `iptables` and easier to manage and use.

1. Install `ufw`

```bash
sudo apt-get install ufw
```

2. Enable the HTTP and HTTPS ports

```bash
sudo ufw allow 80
sudo ufw allow 443
```

3. (Optional) Enabled the SSH port. It's recommended that you set the port to another port that is not `22`, and make sure you update ` /etc/ssh/sshd_config` to match the new port

```bash
sudo ufw allow 22
```

4. Start `ufw`

```bash
sudo ufw enable
sudo ufw status # Should show "Status: active"
```

# 3. Set up a reverse proxy with Caddy

Now that we have ArchiveBox running at `localhost:8000`, we want to publish it as a public-trusted site over HTTPS. We use Caddy which will handle reverse proxy and SSL termination with very little configuration.

1. Set your domain's A record (e.g. `archivebox.example.com`) point to your Vultr server in your DNS provider. Verify correct records with an authoritative lookup

```bash
curl "https://cloudflare-dns.com/dns-query?name=archivebox.example.com&type=A" -H "accept: application/dns-json"
```

2. Install Caddy. There are [many ways](https://caddyserver.com/docs/install) and you can even extend the `docker-compose.yml` of ArchiveBox. Here we will use the standard Caddy package.

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee -a /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

2. Update Caddy's configuration `/etc/caddy/Caddyfile`

```
archivebox.example.com

reverse_proxy localhost:8000
```

3. Run `caddy reload` to reload the configuration gracefully (without downtime)

4. Verify it works by visiting `archivebox.example.com`

# Conclusion

At this point, you will have a self-hosted ArchiveBox application that you can bookmark websites from everywhere!
