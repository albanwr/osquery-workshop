# Install OSQuery
To install osquery on Ubuntu, you can run the following:

```
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $OSQUERY_KEY
sudo add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
sudo apt-get update
sudo apt-get install osquery
```

# Capture enrollment secret & cert
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

Upload the certificate to the target host:

```
scp XXXX sysadmin@<hostname>:
```

If you do not have access to a ssh client, you can also upload the file via the web console in CloudShare.

![upload](https://github.com/sophos-cybersecurity/osquery-workshop/raw/master/images/1.png)

Persist and start the service:
```
sudo systemctl enable osquery
sudo systemctl start osquery
```

Open the osquery command line interface:

```
osqueryi
```