---
title: Docker
description: Getting started with the Docker image
published: true
date: 2020-02-15T05:02:47.376Z
tags: setup, docker
---

> Before proceeding, make sure you meet the [system requirements](/install/requirements).
{.is-info}

# Using the Docker image

Wiki.js is published as a Docker image on Docker Hub as `requarks/wiki`

> It's highly recommended that you don't use the `latest` tag but instead the major version you need, e.g. `requarks/wiki:2`
>
> It's also possible to point to a specific minor version (e.g. `requarks/wiki:2.0`), although you will not automatically get the latest features when pulling the latest image.
{.is-info}

## Environment Variables
You must set the following environment variables. They are all **required** unless specified otherwise.

### Database

*For PostgreSQL, MySQL, MariaDB and MSSQL only:*

- **DB_TYPE** : Type of database (`mysql`, `postgres`, `mariadb`, `mssql` or `sqlite`)
- **DB_HOST** : Hostname or IP of the database
- **DB_PORT** : Port of the database
- **DB_USER** : Username to connect to the database
- **DB_PASS** : Password to connect to the database
- **DB_NAME** : Database name
{.grid-list}

*When connecting to a database server with SSL enforced:*

- **DB_SSL** : Set to either `1` or `true` to enable. *(optional, off if omitted)*
{.grid-list}

*Alternative way to provide the database password, via a local file secret:*

- **DB_PASS_FILE**: Path to the mapped file containing to the database password. *(optional, replaces DB_PASS)*
{.grid-list}

*For SQLite only:*

- **DB_FILEPATH** : Path to the SQLite file
{.grid-list}

### HTTPS

> This feature is only available from version **2.1 and up**
{.is-info}

Environment variables are provided for easy Let's Encrypt configuration.
If you want to provide your own SSL certificate configuration, you must instead mount a config file [as explained below](#alternative-mount-the-config-file).

- **SSL_ACTIVE** : Set to either `1` or `true` to enable. *(optional, off if omitted)*
- **LETSENCRYPT_DOMAIN** : The domain / sub-domain to use when requesting a certificate from Let's Encrypt (e.g. `wiki.example.com`)
- **LETSENCRYPT_EMAIL** : The administrator email used when requesting a certificate from Let's Encrypt.
{.grid-list}

The exposed HTTPS port is `3443`. Both HTTP and HTTPS ports must be exposed when using Let's Encrypt, e.g.:
```bash
docker run -d -p 80:3000 -p 443:3443 -e "LETSENCRYPT_DOMAIN=wiki.example.com" -e "LETSENCRYPT_EMAIL=admin@example.com" --name wiki --restart unless-stopped -e "DB_TYPE=postgres" -e "DB_HOST=db" -e "DB_PORT=5432" -e "DB_USER=wikijs" -e "DB_PASS=wikijsrocks" -e "DB_NAME=wiki" requarks/wiki:2
```

> **The HTTP port must be accessible from the internet for the certificate provisioning to complete!**
> Once the certificate is generated, you can enable automatic HTTPS redirection from the **Administration** area, under the **SSL** section.
> 
> Note that you must leave the HTTP port open and accessible at all times for the certificate renewal process to work. This is **NOT** a security risk when the above option (HTTPS Redirection) is enabled.
{.is-warning}

## Example

Here's an example of a command to run Wiki.js connecting to a PostgreSQL database:
```bash
docker run -d -p 8080:3000 --name wiki --restart unless-stopped -e "DB_TYPE=postgres" -e "DB_HOST=db" -e "DB_PORT=5432" -e "DB_USER=wikijs" -e "DB_PASS=wikijsrocks" -e "DB_NAME=wiki" requarks/wiki:2
```

or to a MySQL database:
```bash
docker run -d -p 8080:3000 --name wiki --restart unless-stopped -e "DB_TYPE=mysql" -e "DB_HOST=db" -e "DB_PORT=3306" -e "DB_USER=wikijs" -e "DB_PASS=wikijsrocks" -e "DB_NAME=wiki" requarks/wiki:2
```

> Both examples assume you have a database running in another container named `db` on the same network.
> Wiki.js does **NOT** come with a database engine. See the [requirements](/install/requirements) for more details.
{.is-warning}

## Alternative: Mount the config file

If using environment variables is not your cup of tea, you can also mount a config file instead.

Create a new file based on the [sample config file](https://github.com/Requarks/wiki/blob/master/config.sample.yml) and modify the values to match your setup. You can then mount the config file in the container:

```bash
docker run -d -p 8080:3000 --name wiki --restart unless-stopped -v YOUR-FILE.yml:/wiki/config.yml requarks/wiki:2
```

It's also possible to define an alternate location for the config file to be loaded from. This is useful in scenarios where you want to mount a configuration folder instead. Define the environment variable **CONFIG_FILE** with the path to the config file.

## Change User Mode

By default, Wiki.js runs as user `wiki`. If you get permissions issues while mounting files (such as SQLite db or private keys), you can override the runtime user to run as `root` using the `-u` parameter, e.g.:

```bash
docker run -d -p 8080:3000 -u="root" --name wiki --restart unless-stopped -e "DB_TYPE=postgres" -e "DB_HOST=db" -e "DB_PORT=5432" -e "DB_USER=wikijs" -e "DB_PASS=wikijsrocks" -e "DB_NAME=wiki" requarks/wiki:2
```

This is however not a secure way to run containers. Make sure you understand the security implications before doing so.

# Using Docker Compose

Here's a full example of a Docker Compose file for Wiki.js, using PostgreSQL, listening on port 80:

```yaml
version: "3"
services:

  db:
    image: postgres:11-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    logging:
      driver: "none"
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "80:3000"

volumes:
  db-data:
```

![](https://a.icons8.com/jihZbhdR/4WJoF7/svg.svg){.align-abstopright}

To update wiki.js to the latest version, use the relevant docker-compose command:
``` bash
docker-compose up -d --no-deps --build wiki
```

# ARM images

In order to use the ARM64 / ARMv7 image variants, you must use the image tag `requarks/wiki:2.1-arm`.

This image is compatible with:

- **ARM64**: Raspberry Pi 4, 3 and later Raspberry Pi 2 (v1.2)
- **ARMv7**: Early Raspberry Pi 2 (v1.1)

The original, first-generation Raspberry Pi is **not** supported *(ARMv6)*.

# OpenShift

An example OpenShift-compatible Dockerfile is available [here](https://github.com/Requarks/wiki/blob/master/dev/openshift/Dockerfile).
