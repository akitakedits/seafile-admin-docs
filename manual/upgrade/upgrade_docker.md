# Upgrade Seafile Docker

For maintenance upgrade, like from version 10.0.1 to version 10.0.4, just download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version, then start with docker compose up.

For major version upgrade, like from 10.0 to 11.0, see instructions below.

Please check the **upgrade notes** for any special configuration or changes before/while upgrading.


## Upgrade from 11.0 to 12.0

From Seafile Docker 12.0, we recommend that you use `.env` and `seafile-server.yml` files for configuration.

### Backup the original docker-compose.yml file:

```sh
mv docker-compose.yml docker-compose.yml.bak
```

### Download the Docker files for Seafile 12.0

Download [.env](../docker/ce/env), [seafile-server.yml](../docker/ce/seafile-server.yml) and [caddy.yml](../docker/ce/caddy.yml), and modify .env file according to the old configuration in `docker-compose.yml.bak`

=== "Seafile pro edition"

    ```sh
    wget -O .env https://manual.seafile.com/12.0/docker/pro/env
    wget https://manual.seafile.com/12.0/docker/pro/seafile-server.yml
    wget https://manual.seafile.com/12.0/docker/pro/caddy.yml
    ```

=== "Seafile community edition"

    ```sh
    wget -O .env https://manual.seafile.com/12.0/docker/ce/env
    wget https://manual.seafile.com/12.0/docker/ce/seafile-server.yml
    wget https://manual.seafile.com/12.0/docker/ce/caddy.yml
    ```

### Configurate Seafile

For the fields in `.env`, please refer to [here](../setup/single_node_installation.md#downloading-and-modifying-docker-files).

SSL is now handled by the [caddy server](../setup/caddy.md). If you have used SSL before, you will also need modify the seafile.nginx.conf. Change server listen 443 to 80.

Backup the original seafile.nginx.conf file:

```sh
cp seafile.nginx.conf seafile.nginx.conf.bak
```

Remove the `server listen 80` section:

```config
#server {
#    listen 80;
#    server_name _ default_server;

    # allow certbot to connect to challenge location via HTTP Port 80
    # otherwise renewal request will fail
#    location /.well-known/acme-challenge/ {
#        alias /var/www/challenges/;
#        try_files $uri =404;
#    }

#    location / {
#        rewrite ^ https://example.seafile.com$request_uri? permanent;
#    }
#}
```

Change `server listen 443` to `80`:

```config
server {
#listen 443 ssl;
listen 80;

#    ssl_certificate      /shared/ssl/pkg.seafile.top.crt;
#    ssl_certificate_key  /shared/ssl/pkg.seafile.top.key;

#    ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;

   ...
```

Start with docker compose up.

### Upgrade SeaDoc from 0.8 to 1.0 for Seafile v12.0

If you have deployed SeaDoc v0.8 with Seafile v11.0, you can upgrade it to 1.0 use the following two steps:

1. Delete sdoc_db.
2. Remove SeaDoc configs in seafile.nginx.conf file.
3. Re-deploy SeaDoc server. In other words, delete the old SeaDoc deployment and deploy a new SeaDoc server on a separate machine.

Note, deploying SeaDoc and **Seafile binary package** on the same server is no longer supported. If you really want to deploying SeaDoc and Seafile server on the same machine, you should deploy Seafile server with Docker.

#### Delete sdoc_db

From version 1.0, SeaDoc is using seahub_db database to store its operation logs and no longer need an extra database sdoc_db. The database tables in seahub_db are created automatically when you upgrade Seafile server from v11.0 to v12.0. You can simply delete sdoc_db.

#### Remove SeaDoc configs in seafile.nginx.conf file

If you have deployed SeaDoc older version, you should remove `/sdoc-server/`, `/socket.io` configs in seafile.nginx.conf file.

```config
#    location /sdoc-server/ {
#        add_header Access-Control-Allow-Origin *;
#        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
#        add_header Access-Control-Allow-Headers "deviceType,token, authorization, content-type";
#        if ($request_method = 'OPTIONS') {
#            add_header Access-Control-Allow-Origin *;
#            add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
#            add_header Access-Control-Allow-Headers "deviceType,token, authorization, content-type";
#            return 204;
#        }
#        proxy_pass         http://sdoc-server:7070/;
#        proxy_redirect     off;
#        proxy_set_header   Host              $host;
#        proxy_set_header   X-Real-IP         $remote_addr;
#        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
#        proxy_set_header   X-Forwarded-Host  $server_name;
#        proxy_set_header   X-Forwarded-Proto $scheme;
#        client_max_body_size 100m;
#    }
#    location /socket.io {
#        proxy_pass http://sdoc-server:7070;
#        proxy_http_version 1.1;
#        proxy_set_header Upgrade $http_upgrade;
#        proxy_set_header Connection 'upgrade';
#        proxy_redirect off;
#        proxy_buffers 8 32k;
#        proxy_buffer_size 64k;
#        proxy_set_header X-Real-IP $remote_addr;
#        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#        proxy_set_header Host $http_host;
#        proxy_set_header X-NginX-Proxy true;
#    }
```

#### Deploy a new SeaDoc server

Please see the document [Setup SeaDoc](../extension/setup_seadoc.md) to install SeaDoc on a separate machine and integrate with your binary packaged based Seafile server v12.0.


## Upgrade from 10.0 to 11.0

Download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version. Taking the [community edition](../setup/single_node_installation.md) as an example, you have to modify

```yml
...
service:
    ...
    seafile:
        image: seafileltd/seafile-mc:10.0-latest
        ...
    ...
```

to

```yml
service:
    ...
    seafile:
        image: seafileltd/seafile-mc:11.0-latest
        ...
    ...
```

 It is also recommended that you upgrade **mariadb** and **memcached** to newer versions as in the v11.0 docker-compose.yml file. Specifically, in version 11.0, we use the following versions:

- MariaDB: 10.11
- Memcached: 1.6.18

What's more, you have to migrate configuration for LDAP and OAuth according to [here](upgrade_notes_for_11.0.x.md)

Start with docker compose up.

## Upgrade from 9.0 to 10.0

Just download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version, then start with docker compose up.

If you are using pro edition with ElasticSearch, SAML SSO and storage backend features, follow the upgrading manual on how to update the configuration for these [features](upgrade_notes_for_10.0.x.md).

If you want to use the new notification server and rate control (pro edition only), please refer to the [upgrading manual](upgrade_notes_for_10.0.x.md).

## Upgrade from 8.0 to 9.0

Just download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version, then start with docker compose up.

### Let's encrypt SSL certificate

Since version 9.0.6, we use Acme V3 (not acme-tiny) to get certificate.

If there is a certificate generated by an old version, you need to back up and move the old certificate directory and the seafile.nginx.conf before starting.

```shell
mv /opt/seafile/shared/ssl /opt/seafile/shared/ssl-bak

mv /opt/seafile/shared/nginx/conf/seafile.nginx.conf /opt/seafile/shared/nginx/conf/seafile.nginx.conf.bak
```

Starting the new container will automatically apply a certificate.

```shell
docker compose down
docker compose up -d
```

Please wait a moment for the certificate to be applied, then you can modify the new seafile.nginx.conf as you want. Execute the following command to make the nginx configuration take effect.

```sh
docker exec seafile nginx -s reload
```

A cron job inside the container will automatically renew the certificate.

## Upgrade from 7.1 to 8.0

Just download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version, then start with docker compose up.

## Upgrade from 7.0 to 7.1

Just download the new image, stop the old docker container, modify the Seafile image version in docker-compose.yml to the new version, then start with docker compose up.

