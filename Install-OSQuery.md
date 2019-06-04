# Install OSQuery with Kolide
The following guide covers the installation of osquery and the configuration of the osquery daemon to connect it back to a Kolide fleet instance.

```
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $OSQUERY_KEY
sudo add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
sudo apt-get update
sudo apt-get install osquery
```

## 1. Capture the Enrollment Details
-  Navigate to:
```
https://<KOLIDE_HOSTNAME>:8080/hosts/manage
```
- Click:
```
Add New Host
```
- Take note of the `Enroll Secret`. You will need it.

![upload](https://github.com/sophos-cybersecurity/osquery-workshop/raw/master/images/2.png)

- Download and store the certificate. Click:

```
Fletch Field Certificate
```
It will download a PEM file with a name based upon your kolide hostname. e.g. `uvo1cddxern2ff8nwkl.vm.cld.sr_8080.pem`

## 2. Upload certificate to target host

We need to upload the certificate to the target host before we can configure the OSQuery daemon. You have two options here:

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

## 3. Configure OSQuery daemon

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

Note: you will need to repeat steps 2 and 3 to each host you intend to connect to osquery.

Logs are available in `/var/log/osquery/*` for troubleshooting.