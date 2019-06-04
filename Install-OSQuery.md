# Install OSQuery with Kolide
To install osquery on Ubuntu, you can run the following:

```
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $OSQUERY_KEY
sudo add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
sudo apt-get update
sudo apt-get install osquery
```

## Capture enrollment secret & cert
-  Navigate to:
```
https://<KOLIDE_HOSTNAME>:8080/hosts/manage
```
- Click:
```
Add New Host
```
- Take note of the `Enroll Secret`. You will need it.

Download and store the certificate:

- Click `Fletch Field Certificate`.

## Upload certificate to target host

### Option 1

SCP the certificate file to the target host:

```
scp <KOLIDE_HOSTNAME>_8080.pem sysadmin@<hostname>:
```

Copy the certificate to the osquery config directory:

```
sudo cp <KOLIDE_HOSTNAME>_8080.pem /var/osquery/server.pem
```

### Option 2
If you do not have access to a ssh client, you can also upload the file via the web console in CloudShare e.g:

![upload](https://github.com/sophos-cybersecurity/osquery-workshop/raw/master/images/1.png)

The resulting files will be uploaded to `/home/sysadmin/Uploads/`.

Copy the certificate to the osquery config directory:

```
sudo cp Uploads/<KOLIDE_HOSTNAME>_8080.pem /var/osquery/server.pem
```

## Configure OSQuery daemon

Store the `enrollment secret` on disk. It will look something like this:

```echo 'LQWzGg9+/yaxxcBUMY7VruDGsJRYULw8' | sudo tee /var/osquery/enroll_secret```

Create the OSQuery daemon config. Note you will need to replace `<KOLIDE_HOSTNAME>`.
```
sudo bash -c 'cat << EOF > /etc/osquery/osquery.conf
{
"options": {
  "enroll_secret_path": "/var/osquery/enroll_secret",
  "tls_server_certs": "/var/osquery/server.pem",
  "tls_hostname":"<KOLIDE_HOSTNAME>:8080",
  "host_identifier": "hostname",
  "enroll_tls_endpoint": "/api/v1/osquery/enroll",
  "config_plugin": "tls",
  "config_tls_endpoint": "/api/v1/osquery/config",
  "config_refresh": 10,
  "disable_distributed": false,
  "distributed_plugin": "tls",
  "distributed_interval": 3,
  "distributed_tls_max_attempts": 3,
  "distributed_tls_read_endpoint": "/api/v1/osquery/distributed/read",
  "distributed_tls_write_endpoint": "/api/v1/osquery/distributed/write",
  "logger_plugin": "tls",
  "logger_tls_endpoint": "/api/v1/osquery/log",
  "logger_tls_period": 10
 }
}
EOF'
```

Persist and start the service:
```
sudo systemctl enable osqueryd
sudo systemctl start osqueryd
```

If everything is okay, you should see your host populate into the Kolide dashboard.
