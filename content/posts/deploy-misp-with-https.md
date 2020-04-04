---
title: "Deploy Your Own Local MISP with HTTPS Supported by mkcert"
date: 2019-01-19T16:13:00+07:00
author: "P. Boonyakarn"
description: "In this short tutorial, I will walk through the steps to integrate SSL/TLS into Malware Intelligence Sharing Platform (MISP) with mkcert by Filippo Valsorda. To make it more simple, I will use docker version of MISP available here as an example."
---

In this short tutorial, I will walk through the steps to integrate SSL/TLS into Malware Intelligence Sharing Platform (MISP) with mkcert by Filippo Valsorda. To make it more simple, I will use docker version of MISP available here as an example.

## Generate a Valid Certificate with mkcert

mkcert's [pre-built binaries](https://github.com/FiloSottile/mkcert/releases) are available on various platforms, choose one that match your client operating system. Some mkcert option will not work if `libnss3-tools` or `nss-tools` is not available on the system. Make sure you have already installed one of them based on you current distribution, in case you're using Linux.

Install local CA and generate locally-trusted certificate can be done by:

```sh
$ ./mkcert -install
Created a new local CA at "/home/user/.local/share/mkcert" 💥
The local CA is now installed in the Firefox and/or Chrome/Chromium trust store (requires browser restart)! 🦊

$ ./mkcert misp.local localhost 127.0.0.1 ::1
Using the local CA at "/home/user/.local/share/mkcert" ✨

Created a new certificate valid for the following names 📜
 - "misp.local"
 - "localhost"
 - "127.0.0.1"
 - "::1"

The certificate is at "./misp.local+3.pem" and the key at "./misp.local+3-key.pem" ✅
```

The commands above will create a certificate and a key for `misp.local`, `localhost`, `127.0.0.1` and `::1`. You can change your MISP domain to whatever you want. The domain `misp.local` will be added to the system's host file to make it easier to access the instance.

## Setup MISP

Manually install MISP according to the installation guidelines is time consuming, and if you want to make multi-purpose environment clean as much as you can, try [MISP docker](https://github.com/MISP/misp-docker). But first, make sure you have already installed git and docker.

Clone MISP docker upstream with git clone:

```sh
$ git clone --depth=1 https://github.com/MISP/misp-docker.git
Cloning into 'misp-docker'...
remote: Enumerating objects: 17, done.
remote: Counting objects: 100% (17/17), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 17 (delta 1), reused 12 (delta 1), pack-reused 0
Unpacking objects: 100% (17/17), done.
```

MISP docker has three containers proxy, misp_db and misp_web, but in this tutorial I will spin on only misp_web and misp_db without proxy. So, I will let you finish some configuration in `docker-compose.yml` files by yourself, including:

- `MYSQL_PASSWORD`
- `MYSQL_ROOT_PASSWORD`
- `MISP_ADMIN_EMAIL` (There's known issue. If this email doesn't work on login, try `admin@admin.test`)
- `MISP_ADMIN_PASSPHRASE` (There's known issue. If this email doesn't work on login, try `admin`)
- `MISP_BASEURL` (mine is `https://misp.local`)
- `TIMEZONE`

**Don't forget to add bind port on 443:443 for SSL/TLS connection**. Because we are going not to use proxy, we have to specify docker-compose file we are going to build:

```sh
$ cd misp-docker/
$ docker-compose -f docker-compose.yml build
...
...
...
<wait until building process is done>
$ docker-compose -f docker-compose.yml up
Creating misp_db  ... done
Creating misp_web ... done
Attaching to misp_db, misp_web
misp_db | [Entrypoint] MySQL Docker Image 5.7.24-1.1.8
misp_db | [Entrypoint] Initializing database
misp_web | Container started for the fist time. Setup might time a few minutes. Please wait...
misp_web | (Details are logged in /tmp/install.log)
misp_web | Restoring MISP files...
misp_web | Configuring postfix
...
```

Because the default configuration for apache2 MISP relies on doesn't have SSL/TLS function enable. We need to enable manually by spawning shell on container and issue the following command:

```sh
$ docker exec -ti misp_web bash
root@2e215f3c031d:/etc/apache2# a2enmod ssl
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Enabling module socache_shmcb.
Enabling module ssl.
See /usr/share/doc/apache2/README.Debian.gz on how to configure SSL and create self-signed certificates.
To activate the new configuration, you need to run:
  service apache2 restart
root@2e215f3c031d:/etc/apache2#
```

To make MISP available on TCP/443 for supported SSL/TLS connection, update the existing apache2 configuration on `/etc/apache2/sites-enabled/misp.conf`

You can simply update the configuration and copy certificate and key to the container with docker cp command:

```sh
$ docker cp misp.conf misp_web:/etc/apache2/sites-enabled/
$ docker cp misp.local+3.pem misp_web:/etc/ssl/private/misp.crt
$ docker cp misp.local+3-key.pem misp_web:/etc/ssl/private/misp.key
```

Get back inside the container and restart apache2 service with `service apache2 restart`.

Update your host file to add misp.local for MISP server IP address and try accessing with `https://misp.local` and don't forget to update the instance `MISP.baseurl` configuration on `https://misp.local/servers/serverSettings/MISP`  to be `https://misp.local`