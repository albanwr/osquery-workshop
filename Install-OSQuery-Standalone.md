# Install OSQuery
To install osquery on Ubuntu, you can run the following:

```
export OSQUERY_KEY=1484120AC4E9F8A1A577AEEE97A80C63C9D8B80B
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $OSQUERY_KEY
sudo add-apt-repository 'deb [arch=amd64] https://pkg.osquery.io/deb deb main'
sudo apt-get update
sudo apt-get install osquery
```

Persist and start the service:
```
sudo systemctl enable osquery
sudo systemctl start osquery
```

Open the osquery command line interface:

```
osqueryi
```