Kolide Fleet on Ubuntu
======================

In this guide, we're going to install Kolide Fleet and all of it's application dependencies on an Ubuntu 16.04 LTS server. Once we have Fleet up and running, we're going to install osquery on that same Ubuntu 16.04 host and enroll it in Fleet. This should give you a good understanding of both how to install Fleet as well as how to install and configure osquery such that it can communicate with Fleet.

## Installing Fleet

To install Fleet, run the following:

```
wget https://dl.kolide.co/bin/fleet_latest.zip
unzip fleet_latest.zip 'linux/*' -d fleet
sudo cp fleet/linux/fleet /usr/bin/fleet
sudo cp fleet/linux/fleetctl /usr/bin/fleetctl
```

## Installing and configuring dependencies

### MySQL & Redis

To install the MySQL server and redis, run the following:

```
sudo apt-get install mysql-server redis-server -y
```

When asked for MySQL's root password, enter `toor` for the sake of this tutorial if you are having trouble thinking of a better password for the MySQL root user. If you decide to set your own password, be mindful that you will need to substitute it every time `toor` is used in this document.

```
sudo systemctl enable mysql.service
sudo systemctl start mysql.service
sudo systemctl enable redis-server.service
sudo systemctl start redis-server.service
```

It's also worth creating a MySQL database for us to use at this point. Run the following to create the `kolide` database in MySQL. Note that you will be prompted for the password you created above.

```
$ echo 'CREATE DATABASE kolide;' | mysql -u root -p
```

## Running the Fleet server

Now that we have installed Fleet, MySQL, and Redis, we are ready to launch Fleet! First, we must "prepare" the database. We do this via `fleet prepare db`:

```
$ /usr/bin/fleet prepare db \
    --mysql_address=127.0.0.1:3306 \
    --mysql_database=kolide \
    --mysql_username=root \
    --mysql_password=toor
```

The output should look like:

`Migrations completed`

Before we can run the server, we need to generate some TLS keying material. If you already have tooling for generating valid TLS certificates, then you are encouraged to use that instead. You will need a TLS certificate and key for running the Fleet server. If you'd like to generate self-signed certificates, you can do this via the following steps (note - you will be asked for severl bits of information, including name, contact info, and location, in order to generate the certificate):

```
sudo mkdir /etc/kolide
sudo openssl genrsa -out /etc/kolide/server.key 4096
sudo openssl req -new -key /etc/kolide/server.key -out /etc/kolide/server.csr
sudo openssl x509 -req -days 366 -in /etc/kolide/server.csr -signkey /etc/kolide/server.key -out /etc/kolide/server.cert
```

You should now have three new files in `/etc/kolide`:

- `server.cert`
- `server.key`
- `server.csr`

Now we are ready to run the server! We do this via `fleet serve`:

```
$ /usr/bin/fleet serve \
  --mysql_address=127.0.0.1:3306 \
  --mysql_database=kolide \
  --mysql_username=root \
  --mysql_password=toor \
  --redis_address=127.0.0.1:6379 \
  --server_cert=/etc/kolide/server.cert \
  --server_key=/etc/kolide/server.key \
  --logging_json
```

You will be prompted to add a value for `--auth_jwt_key`. A randomly generated key will be suggested, you can simply add the flag with the sugested key and run the command again.

Now, if you go to [https://<CLOUDSHARE_HOSTNAME>:8080](https://localhost:8080) in your local browser, you should be redirected to [https://<CLOUDSHARE_HOSTNAME>:8080/setup](https://localhost:8080/setup) where you can create your first Fleet user account.

