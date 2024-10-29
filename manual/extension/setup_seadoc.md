# SeaDoc Integration

SeaDoc is an extension of Seafile that providing an online collaborative document editor.

SeaDoc designed around the following key ideas:

* An expressive easy to use editor
* A review and approval workflow to better control how contents changes
* Inter-document linking for connecting related contents
* AI integration that streamlines content generation, summarization, and management
* Comprehensive APIs for automating document generating and processing

SeaDoc excels at:

* Authoring product and technical documents
* Creating knowledge base articles and online manuals
* Building internal Wikis

## Architecture

The SeaDoc archticture is demonstrated as below:

![SeaDoc](../images/seadoc-arch.png)

Here is the workflow when a user open sdoc file in browser

1. When a user open a sdoc file in the browser, a file loading request will be sent to Caddy, and Caddy proxy the request to SeaDoc server (see [Seafile instance archticture](../setup/overview.md) for the details).
2. SeaDoc server will send the file's content back if it is already cached, otherwise SeaDoc serve will sends a request to Seafile server.
3. Seafile server loads the content, then sends it to SeaDoc server and write it to the cache at the same time.
4. After SeaDoc receives the content, it will be sent to the browser.

## Deployment method

SeaDoc has the following deployment methods:

- [SeaDoc and Seafile docker are deployed on the same host](#seadoc-and-seafile-docker-are-deployed-on-the-same-host).
- [Deploy SeaDoc on a new host](#deploy-seadoc-on-a-new-host).

## SeaDoc and Seafile docker are deployed on the same host

Download the `seadoc.yml` and integrate SeaDoc in Seafile docker.

=== "Seafile Pro Edition"
    ```sh
    wget https://manual.seafile.com/12.0/docker/pro/seadoc.yml
    ```
=== "Seafile Community Edition"
    ```sh
    wget https://manual.seafile.com/12.0/docker/ce/seadoc.yml
    ```

Modify `.env`, and insert `seadoc.yml` into `COMPOSE_FILE`, and enable SeaDoc server

```shell
COMPOSE_FILE='seafile-server.yml,caddy.yml,seadoc.yml'

ENABLE_SEADOC=true
SEADOC_SERVER_URL=https://example.seafile.com/sdoc-server
```

Start SeaDoc server with the following command

```sh
docker compose up -d
```

Now you can use SeaDoc!

## Deploy SeaDoc on a new host

Download and modify the `.env` and `seadoc.yml` files.

Download [seadoc.yml](../docker/seadoc/1.0/standalone/seadoc.yml) and [.env](../docker/seadoc/1.0/standalone/env) sample files to your host. Then modify the `.env` file according to your environment. The following fields are needed to be specified:

!!! note "Database used by SeaDoc"
    SeaDoc will use one database table `seahub_db.sdoc_operation_log` to store operation logs.

| Variable                  | Description                                                                 | Default Value   |  
| ------------------------- | ----------------------------------------------------------------------------- | --------------- |  
| `SEADOC_VOLUME`           | The volume directory of SeaDoc data. This is a required input.                | (required)      |  
| `SEAFILE_MYSQL_DB_HOST`   | The host address of the MySQL database used by Seafile.                       | `db`            |  
| `SEAFILE_MYSQL_DB_USER`   | The username for the MySQL database used by Seafile.                          | `seafile`       |  
| `SEAFILE_MYSQL_DB_PASSWORD` | The password for the MySQL database user used by Seafile. This is a required input. | (required)      |  
| `TIME_ZONE`               | The time zone setting for the system.                                        | `UTC`           |  
| `JWT_PRIVATE_KEY`         | The JWT private key, which should match the config in the Seafile `.env` file. | (required)      |  
| `SEAFILE_SERVER_HOSTNAME` | The host name of the Seafile server. This is a required input.                | (required)      |  
| `SEAFILE_SERVER_PROTOCOL` | The protocol used to access the Seafile server (http or https).               | `http`          |  
| `SEADOC_SERVER_URL`       | The URL of the SeaDoc service. This is a required input.                      | (required)      |

Start SeaDoc server with the following command

```sh
docker compose up -d
```

Then bind SeaDoc server url and ip in the load balance(or reverse proxy) configuration.

Now you can use SeaDoc!

## SeaDoc directory structure

### `/opt/seadoc-data`

Placeholder spot for shared volumes. You may elect to store certain persistent information outside of a container, in our case we keep various log files outside. This allows you to rebuild containers easily without losing important information. SeaDoc logs will be storaged in `/opt/seadoc-data/logs`.
