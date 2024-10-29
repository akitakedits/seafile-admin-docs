# Migrate From SQLite to MySQL

!!! tip "The tutorial is only related to Seafile CE edition"

First make sure the python module for MySQL is installed. On Ubuntu/Debian, use `sudo apt-get install python-mysqldb` or `sudo apt-get install python3-mysqldb` to install it.

Steps to migrate Seafile from SQLite to MySQL:

Stop Seafile and Seahub.

Download [sqlite2mysql.sh](https://raw.githubusercontent.com/haiwen/seahub/master/scripts/sqlite2mysql.sh) and [sqlite2mysql.py](https://raw.githubusercontent.com/haiwen/seahub/master/scripts/sqlite2mysql.py) to the top directory of your Seafile installation path. For example, `/opt/seafile`.

Run `sqlite2mysql.sh`:

```
chmod +x sqlite2mysql.sh
./sqlite2mysql.sh
```

This script will produce three files: `ccnet-db.sql`, `seafile-db.sql`, `seahub-db.sql`.

Then create 3 databases ccnet_db, seafile_db, seahub_db and seafile user.

```
mysql> create database ccnet_db character set = 'utf8';
mysql> create database seafile_db character set = 'utf8';
mysql> create database seahub_db character set = 'utf8';
```

Import ccnet data to MySql.

```
mysql> use ccnet_db;
mysql> source ccnet-db.sql;
```

Import seafile data to MySql.

```
mysql> use seafile_db;
mysql> source seafile-db.sql;
```

Import seahub data to MySql.

```
mysql> use seahub_db;
mysql> source seahub-db.sql;
```

Modify configure files：Append following lines to [ccnet.conf](../config/ccnet-conf.md):

```
[Database]
ENGINE=mysql
HOST=127.0.0.1
PORT = 3306
USER=root
PASSWD=root
DB=ccnet_db
CONNECTION_CHARSET=utf8
```

!!! warning "Use `127.0.0.1`, don't use `localhost`"

Replace the database section in `seafile.conf` with following lines:

```
[database]
type=mysql
host=127.0.0.1
port = 3306
user=root
password=root
db_name=seafile_db
connection_charset=utf8
```

Append following lines to `seahub_settings.py`:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'USER' : 'root',
        'PASSWORD' : 'root',
        'NAME' : 'seahub_db',
        'HOST' : '127.0.0.1',
        'PORT': '3306',
        # This is only needed for MySQL older than 5.5.5.
        # For MySQL newer than 5.5.5 INNODB is the default already.
        'OPTIONS': {
            "init_command": "SET storage_engine=INNODB",
        }
    }
}
```

Restart seafile and seahub

!!! note "Note"

    User notifications will be cleared during migration due to the slight difference between MySQL and SQLite, if you only see the busy icon when click the notitfications button beside your avatar, please remove `user_notitfications` table manually by:

    ```
    use seahub_db;
    delete from notifications_usernotification;
    ```

## FAQ

#### Encountered `errno: 150 "Foreign key constraint is incorrectly formed"`

This error typically occurs because the current table being created contains a foreign key that references a table whose primary key has not yet been created. Therefore, please check the database table creation order in the SQL file. The correct order is:

```
auth_user
auth_group
auth_permission
auth_group_permissions
auth_user_groups
auth_user_user_permissions
```
and
```
post_office_emailtemplate
post_office_email
post_office_attachment
post_office_attachment_emails
```
