# Installation of Seafile Server Professional Edition with Docker

This manual explains how to deploy and run Seafile Server Professional Edition (Seafile PE) on a Linux server using Docker and Docker Compose. The deployment has been tested for Debian/Ubuntu and CentOS, but Seafile PE should also work on other Linux distributions.

## Requirements

Seafile PE requires a minimum of 2 cores and 2GB RAM. 

!!! note "Other requirements for Seafile PE"
    If Elasticsearch is installed on the same server, the minimum requirements are 4 cores and 4 GB RAM, and make sure the [mmapfs counts](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-store.html#mmapfs) do not cause excptions like out of memory, which can be increased by following command (see <https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html> for futher details):

    ```shell
    sysctl -w vm.max_map_count=262144 #run as root
    ```

    or modify **/etc/sysctl.conf** and reboot to set this value permanently:

    ```shell
    nano /etc/sysctl.conf

    # modify vm.max_map_count
    vm.max_map_count=262144
    ```

!!! tip "About license"
    Seafile PE can be used without a paid license with up to three users. Licenses for more user can be purchased in the [Seafile Customer Center](https://customer.seafile.com) or contact Seafile Sales at [sales@seafile.com](mailto:sales@seafile.com). For futher details, please refer the [license page](../setup_binary/seafile_professional_sdition_software_license_agreement.md) of Seafile PE.

## Setup

The following assumptions and conventions are used in the rest of this document:

- `/opt/seafile` is the directory of Seafile for storing Seafile docker files. If you decide to put Seafile in a different directory, adjust all paths accordingly.
- Seafile uses two [Docker volumes](https://docs.docker.com/storage/volumes/) for persisting data generated in its database and Seafile Docker container. The volumes' [host paths](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) are /opt/seafile-mysql and /opt/seafile-data, respectively. It is not recommended to change these paths. If you do, account for it when following these instructions.
- All configuration and log files for Seafile and the webserver Nginx are stored in the volume of the Seafile container.

### Installing Docker

Use the [official installation guide for your OS to install Docker](https://docs.docker.com/engine/install/).

### Downloading the Seafile Image

Log into Seafile's private repository and pull the Seafile image:

```bash
docker login docker.seadrive.org
docker pull docker.seadrive.org/seafileltd/seafile-pro-mc:12.0-latest
```

When prompted, enter the username and password of the private repository. They are available on the download page in the [Customer Center](https://customer.seafile.com/downloads).

!!! note
    Older Seafile PE versions are also available in the repository (back to Seafile 7.0). To pull an older version, replace '12.0-latest' tag by the desired version.

### Downloading and Modifying `.env`

From Seafile Docker 12.0, we use `.env`, `seafile-server.yml`  and `caddy.yml` files for configuration.

```bash
mkdir /opt/seafile
cd /opt/seafile

# Seafile PE 12.0
wget -O .env https://manual.seafile.com/12.0/docker/pro/env
wget https://manual.seafile.com/12.0/docker/pro/seafile-server.yml
wget https://manual.seafile.com/12.0/docker/pro/caddy.yml

nano .env
```

The following fields merit particular attention:

| Variable                        | Description                                                                                                   | Default Value                   |  
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------- |  
| `SEAFILE_VOLUME`                | The volume directory of Seafile data                                                                          | `/opt/seafile-data`             |  
| `SEAFILE_MYSQL_VOLUME`          | The volume directory of MySQL data                                                                            | `/opt/seafile-mysql/db`         |  
| `SEAFILE_CADDY_VOLUME`          | The volume directory of Caddy data used to store certificates obtained from Let's Encrypt's                    | `/opt/seafile-caddy`            |  
| `SEAFILE_ELASTICSEARCH_VOLUME`  | (Only valid for Seafile PE) The volume directory of Elasticsearch data | `/opt/seafile-elasticsearch/data` |  
| `INIT_SEAFILE_MYSQL_ROOT_PASSWORD` | The `root` password of MySQL                                                                                  | (required) |  
| `SEAFILE_MYSQL_DB_USER`         | The user of MySQL (`database` - `user` can be found in `conf/seafile.conf`)                                    | `seafile`  |  
| `SEAFILE_MYSQL_DB_PASSWORD`     | The user `seafile` password of MySQL                                                                          | (required)  |  
| `JWT`                           | JWT_PRIVATE_KEY, A random string with a length of no less than 32 characters is required for Seafile, which can be generated by using `pwgen -s 40 1` | (required) |  
| `SEAFILE_SERVER_HOSTNAME`       | Seafile server hostname or domain                                                                  | (required)  |  
| `SEAFILE_SERVER_PROTOCOL`       | Seafile server protocol (http or https)                                                                       | `http` |  
| `TIME_ZONE`                     | Time zone                                                                                                     | `UTC`                           |  
| `INIT_SEAFILE_ADMIN_EMAIL`      | Admin username                                                                                                | me@example.com |  
| `INIT_SEAFILE_ADMIN_PASSWORD`   | Admin password       | asecret |

To conclude, set the directory permissions of the Elasticsearch volumne:

```bash
mkdir -p /opt/seafile-elasticsearch/data
chmod 777 -R /opt/seafile-elasticsearch/data
```

### Starting the Docker Containers

Run docker compose in detached mode:

```bash
docker compose up -d
```

!!! note
    You must run the above command in the directory with the `.env`. If `.env` file is elsewhere, please run

    ```sh
    docker compose -f /path/to/.env up -d
    ```

Wait a few moment for the database to initialize. You can now access Seafile at the host name specified in the Compose file. 

!!! tip "A 502 Bad Gateway error means that the system has not yet completed the initialization"

### Find logs

To view Seafile docker logs, please use the following command

```shell
docker compose logs -f
```

The Seafile logs are under `/shared/logs/seafile` in the docker, or `/opt/seafile-data/logs/seafile` in the server that run the docker.

The system logs are under `/shared/logs/var-log`, or `/opt/seafile-data/logs/var-log` in the server that run the docker.

### Activating the Seafile License

If you have a `seafile-license.txt` license file, simply put it in the volume of the Seafile container. The volumne's default path in the Compose file is `/opt/seafile-data`. If you have modified the path, save the license file under your custom path.

Then restart Seafile:

```bash
docker compose down

docker compose up -d
```

## Seafile directory structure

### Path `/opt/seafile-data`

Placeholder spot for shared volumes. You may elect to store certain persistent information outside of a container, in our case we keep various log files and upload directory outside. This allows you to rebuild containers easily without losing important information.

* /opt/seafile-data/seafile: This is the directory for seafile server configuration 、logs and data.
  * /opt/seafile-data/seafile/logs: This is the directory that would contain the log files of seafile server processes. For example, you can find seaf-server logs in `/opt/seafile-data/seafile/logs/seafile.log`.
* /opt/seafile-data/logs: This is the directory for operating system and Nginx logs.
  * /opt/seafile-data/logs/var-log: This is the directory that would be mounted as `/var/log` inside the container. For example, you can find the nginx logs in `/opt/seafile-data/logs/var-log/nginx/`.

### Reviewing the Deployment

The command `docker container list` should list the containers specified in the `.env`.

The directory layout of the Seafile container's volume should look as follows:

```bash
$ tree /opt/seafile-data -L 2
/opt/seafile-data
├── logs
│   └── var-log
├── nginx
│   └── conf
└── seafile
    ├── ccnet
    ├── conf
    ├── logs
    ├── pro-data
    ├── seafile-data
    └── seahub-data
```

All Seafile config files are stored in `/opt/seafile-data/seafile/conf`. The nginx config file is in `/opt/seafile-data/nginx/conf`.

Any modification of a configuration file requires a restart of Seafile to take effect:

```bash
docker compose restart
```

All Seafile log files are stored in `/opt/seafile-data/seafile/logs` whereas all other log files are in `/opt/seafile-data/logs/var-log`.

## Backup and Recovery

Follow the instructions in [Backup and restore for Seafile Docker](../administration/backup_recovery.md)

## Garbage Collection

When files are deleted, the blocks comprising those files are not immediately removed as there may be other files that reference those blocks (due to the magic of deduplication). To remove them, Seafile requires a ['garbage collection'](../administration/seafile_gc.md) process to be run, which detects which blocks no longer used and purges them.

The required scripts can be found in the `/scripts` folder of the docker container. To perform garbage collection, simply run `docker exec seafile /scripts/gc.sh`.


## FAQ

Q: If I want enter into the Docker container, which command I can use?

A: You can enter into the docker container using the command:

```bash
docker exec -it seafile /bin/bash
```


Q: I forgot the Seafile admin email address/password, how do I create a new admin account?

A: You can create a new admin account by running

```shell
docker exec -it seafile /opt/seafile/seafile-server-latest/reset-admin.sh
```

The Seafile service must be up when running the superuser command.

Q: If, for whatever reason, the installation fails, how do I to start from a clean slate again?

A: Remove the directories /opt/seafile, /opt/seafile-data, /opt/seafile-elasticsearch, and /opt/seafile-mysql and start again.

Q: Something goes wrong during the start of the containers. How can I find out more?

A: You can view the docker logs using this command: `docker compose logs -f`.
