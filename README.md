# Simple Secure Web Downloads

Just a simple pre-configured NGINX server to provide secure downloads with self-signed certs.

Here are some example URLs (that will work **after** the server is started):

- Home: https://HOST
- Downloads directory: https://HOST/downloads
- Direct download for file1.zip: https://HOST/downloads/file1.zip

> Replace `HOST` with your server IP address or domain

By default the server is restricted to the following users:

- User: `demouser`
- Pass: `download1`

See [users section](#users) how to change the default users password and/or add further users.

> SSL certs are self-signed but the default project can easily modified to [get trusted certificates](#trusted-ssl-certs) out of the box.

## Install

1. Install Docker (for example with [this helper script](https://github.com/daverolo/helpers/blob/main/global/installdocker.sh))
1. Fetch the repo to your server
1. Change to the repo root directory
1. Start the server as desribed in the [commands section](#commands)

## Commands

To start the server run:

```
sudo docker compose up -d
```

To stop the server run:

```
sudo docker compose down
```

To restart the server run:

```
sudo docker compose down ; sudo docker compose up -d
```

## Downloads

**All** files inside the [html directory](./html/) are accessible (after the password is entered).
However, it makes sense to put additional file downloads to the [downloads directory](./downloads/).

> Note that files inside the download directory auto-indexed and listed by intention.

## Users

Optional add further users that can access the web root as follows:

1. Create an encrypted password for the new user

```
 openssl passwd -apr1 NEWPASSWORD
```

> Add a space in front of the command as in the example above to avoid the raw password is saved in the shell history!
> Alternatively remove the last command from history via `history -d $(history 1 | awk '{print $1}')'`

2. Open the [htppasswd](./htpasswd) file and add the new user, e.g:

```
mynewuser:$apr1$KXrmgU1s$E7fi4tzAlzjzqMkUtpz.h1
```

## SSL certs

To generate new self-signed SSL certs run:

```
mkdir -p certs &>/dev/null
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout certs/nginx-selfsigned.key -out certs/nginx-selfsigned.crt -subj "/C=AT/ST=Lower Austria/L=Stockerau/O=RockLogic GmbH/OU=DevOps Unit/CN=selfsignedhost"
```

> Optionally change organisation details as needed

## Trusted SSL certs

As mentioned above, SSL certs are self-signed but you can easily expand/modify this project with a domain and an additional proxy like [NginxCrypt](https://github.com/RockLogicGmbH/nginxcrypt) to get trusted certificates by Let's Encrypt out of the box.

To do so you have to follow a few tiny steps.

1. Point your domain in the DNS to the machine where this server is running (note that it can take up to 48H until the domain is available!)
2. Open [docker-compose.nginxcrypt.yaml](./docker-compose.nginxcrypt.yaml) and replace `127.0.0.1` with your actual domain name
3. Stop your current running server via `sudo docker compose down`
4. Rename [nginx.conf](./nginx.conf) to `nginx.original.conf`
5. Rename [docker-compose.yaml](./docker-compose.yaml) to `docker-compose.original.yaml`
6. Rename [nginx.nginxcrypt.conf](./nginx.nginxcrypt.conf) to `nginx.conf`
7. Rename [docker-compose.nginxcrypt.yaml](./docker-compose.nginxcrypt.yaml) to `docker-compose.yaml`
8. Start your server via `sudo docker compose up -d`

Alternatively, if you do not want to rename the files you can also just add `-f docker-compose.nginxcrypt.yaml` to all docker compose commands, like:

```
docker compose -f docker-compose.nginxcrypt.yaml up -d
```

to start the server or to stop it:

```
docker compose -f docker-compose.nginxcrypt.yaml down
```

In such case it is recommended to still rename at least the default `docker-compose.yaml` to a different name to avoid accidentaly start the original NGINX server if you forget to add `-f docker-compose.nginxcrypt.yaml`!

Also get familiar with the [NginxCrypt](https://github.com/RockLogicGmbH/nginxcrypt) config settings to make sure your settings are optimal for production usage.
