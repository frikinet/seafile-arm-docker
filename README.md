# Minimal Seafile on ARM

This repository is based in ChatDeBlofeld/seafile-arm-docker, which is a much complete deployment with a full SQL server and a reverse-proxy in the same device. ALL CREDITS AND THANKS TO ChatDeBlofeld FOR HIS WORK. Here I just have deleted all the unneeded configurations and files to have just a minimal installation of seafile with SQLITE for my own purposes.

A minimal docker-compose based deployment intended to bring seafile on any ARMv7/v8 board with little effort. Just the bare minimum installation for a home seafile server with little users and an external Reverse Proxy in a different device (ej: HAPROXY).

This environment has been set up using the following images and packages (which you should glance at for further documentation):

- [Base Docker image]( https://github.com/frikinet/seafile-arm-docker-base )

No guarantees of any kind are provided by using this environment.

## Prerequisites

A functional docker-compose environment.

## Initialization

Clone this repository in a folder somewhere and change the commented sections in docker-compose.yml and docker-compose.init.yml with the values that fit your case. More configuration - especially for letsencrypt - can be set in the compose file, read the associated documentation if needed.

Then run the following command to begin the installation:

```
docker-compose -f docker-compose.yml -f docker-compose.init.yml run --rm seafile
```

Here are some configurations hints:

```
What is the name of the server? It will be displayed on the client.
3 - 15 letters or digits
[ server name ] Whatever

What is the ip or domain of the server?
For example: www.mycompany.com, 192.168.1.101
[ This server's ip or domain ] Your domain set in the compose file

Which port do you want to use for the seafile fileserver?
[ default "8082" ] Use default

-------------------------------------------------------
Please choose a way to initialize seafile databases:
-------------------------------------------------------

[1] Create new ccnet/seafile/seahub databases
[2] Use existing ccnet/seafile/seahub databases

[ 1 or 2 ] 1

What is the host of mysql server?
[ default "localhost" ] db

From which hosts could the mysql account be used?
[ default "%" ] Use default

What is the port of mysql server?
[ default "3306" ] Use default

What is the password of the mysql root user?
[ root password ] The password you set in the compose file

Enter the name for mysql user of seafile. It would be created if not exists.
[ default "seafile" ] Whatever (default is fine)

Enter the password for mysql user "seafile":
[ password for seafile ] Whatever

Enter the database name for ccnet-server:
[ default "ccnet-db" ]  Whatever (default is fine)

Enter the database name for seafile-server:
[ default "seafile-db" ] Whatever (default is fine)

Enter the database name for seahub:
[ default "seahub-db" ] Whatever (default is fine)
```

Then fill in an admin account and the installation is done.

Finally, run `docker-compose stop` to stop the containers during the next configuration step.

## Configuration

### In `/path/to/seafile/volume/conf/ccnet.conf` 

Enable https:

```
SERVICE_URL = https://your.domain
USE_X_FORWARDED_HOST = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

### In `/path/to/seafile/volume/conf/gunicorn.conf.py`

Change:

`bind = "127.0.0.1:8000"`

To:

`bind = "seafile:8000"`

>Note: for an instance with few users, decrease the number of workers will significantly reduce RAM usage

### In `/path/to/seafile/volume/conf/seahub_settings.py` 

Add the following lines (with your domain correctly set):

```
FILE_SERVER_ROOT = 'https://your.domain/seafhttp'
```

## Run

Simply run:

```
docker-compose up -d
```

You should now be able to access `http://your-ip:your-port` and log in with your previously set admin account.

## Troubleshooting

For any initialization bugs, first try removing all volumes and containers and run again.

### Seahub failed to start

The `docker logs` command could provide precious information about what went wrong.

If seahub did not start, try running new containers with `docker-compose up -d --force-recreate`. Manually start it inside the container with `/opt/seafile/seafile-server-latest/seafile.sh start-fastcgi` provide more logs and could be useful.

It's also possible to set `daemon` to `False` in `gunicorn.conf.py` to grab more information about a failed start.

