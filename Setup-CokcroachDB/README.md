# Setup CockroachDB

## Basic Setup
Setup hostname for all node
```
hostnamectl set-hostaname cockroachdb-1
hostnamectl set-hostaname cockroachdb-2
hostnamectl set-hostaname cockroachdb-3
```

Setup time for synchronize time
```
apt install systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
sudo timedatectl set-timezone Asia/Jakarta
sudo timedatectl set-ntp true
```

Setup /etc/hosts file
```
$ip_address cockroachdb-1
$ip_address cockroachdb-2
$ip_address cockroachdb-3
```

## Installations
Download the binary file cockroachDB, you can see the reference release on this link: www.cockroachlabs.com/docs/releases/
```
wget https://binaries.cockroachdb.com/cockroach-v24.2.4.linux-amd64.tgz
```

Extract the binary file
```
tar -xvzf cockroach-v24.2.4.linux-amd64.tgz
```

Copy to dir bin
```
sudo cp cockroach-*/cockroach /usr/local/bin/
```

Check the installations
```
cockroach version
```

## Create Cluster CockroachDB
