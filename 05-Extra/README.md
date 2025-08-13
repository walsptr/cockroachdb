# Extra

## Setup replica number for database
increase replica number to database
```
ALTER DATABASE training CONFIGURE ZONE USING num_replicas = 5;
```

increase replica number by default config
```
ALTER RANGE default CONFIGURE ZONE USING num_replicas = 5;
```

## Monitoring cockroachdb
by default cockroachdb menyediakan web ui untuk monitoring, dapat diakses melalui $ip_address:8080

## Adding Node to cluster
- setup node baru seperti kita setup awal tadi
- sesuaikan certificate server sesuai hostname dan ip address node yang baru
- sesuaikan systemd pada node yang baru

## Upgrade cockroachdb version
- make sure doing backup for database
- download new version binary
```
wget https://binaries.cockroachdb.com/cockroach-v24.3.3.linux-amd64.tgz
```
- stop service cockroachdb
```
systemctl stop cockroachdb
```
- extract and copy file binary to bin directory
```
tar -xvzf cockroach-v24.3.3.linux-amd64.tgz
cp cockroach-v24.3.3.linux-amd64/cockroach /usr/local/bin/
chown cockroach:cockroach /usr/local/bin/cockroach
cockroach version
```
- start again service cockroachdb
```
systemctl start cockroachdb
```
- check status service
```
systemctl status cockroachdb
```
