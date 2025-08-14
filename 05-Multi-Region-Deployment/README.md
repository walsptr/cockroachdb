# Multi Region Deployment CockroachDB

## Setup cockroachdb1-3
Edit systemd file on cockroachdb-1
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-westsumatra1,zone=ind-westsumatra-1a --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

Edit systemd file on cockroachdb-2
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-eastjava1,zone=ind-eastjava-1a --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

Edit systemd file on cockroachdb-3
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-westjava1,zone=ind-westjava1-1a --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

check regions status
```
SHOW REGIONS FROM CLUSTER;
```

## Setup Cockroachdb4-6
lakukan langkah-langkah seperti pada setup cockroachdb di awal untuk cockroachdb4-6

### Basic Setup
Setup hostname for all node
```
hostnamectl set-hostaname cockroachdb-4
hostnamectl set-hostaname cockroachdb-5
hostnamectl set-hostaname cockroachdb-6
```

Setup time for synchronize time
```
apt install systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
sudo timedatectl set-timezone Asia/Jakarta
sudo timedatectl set-ntp true
```

Setup /etc/hosts file on cockroachdb-1
```
$ip_address cockroachdb-4
$ip_address cockroachdb-5
$ip_address cockroachdb-6
```

Setup sudo no passwd for user
```
sudo vim /etc/sudoers

$user ALL=(ALL:ALL) NOPASSWD: ALL
```

Setup SSH Config
```
vim ~/.ssh/config

Host cockroachdb-4
  HostName cockroachdb-4
  User $user
  StrictHostKeyChecking no
Host cockroachdb-5
  HostName cockroachdb-5
  User $user
  StrictHostKeyChecking no
Host cockroachdb-6
  HostName cockroachdb-6
  User $user
  StrictHostKeyChecking no
```

Setup SSH Public key
```
ssh-copy-id cockroachdb-4
ssh-copy-id cockroachdb-5
ssh-copy-id cockroachdb-6
```

### Installations
Download the binary file cockroachDB on node cockroachdb4-6
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

### Create Cluster CockroachDB

Create work dir and external connection dir cockroachdb on cockroachdb4-6
```
mkdir -p /opt/cockroachdb/certs
mkdir /mnt/cockroachdb
```

Copy CA to cockroachdb4-6
```
scp ~/certs/ca.crt ~/certs/ca.key cockroachdb-4:/tmp
scp ~/certs/ca.crt ~/certs/ca.key cockroachdb-5:/tmp
scp ~/certs/ca.crt ~/certs/ca.key cockroachdb-6:/tmp
```

Move the CA file and key to Work Dir
```
ssh cockroachdb-4
sudo mv /tmp/ca.* /opt/cockroachdb/certs

ssh cockroachdb-5
sudo mv /tmp/ca.* /opt/cockroachdb/certs

ssh cockroachdb-6
sudo mv /tmp/ca.* /opt/cockroachdb/certs
```

Create client cert on cockroachdb4-6
```
cockroach cert create-client root --certs-dir=certs --ca-key=certs/ca.key
```

Setup server cert on all cockroachdb4-6
```
cockroach cert create-node localhost $hostname $ip_address $ip_load_balancer $hostname_load_balancer --certs-dir=certs --ca-key=certs/ca.key
```

Setup user for cockroachdb service on cockroachdb4-6
```
useradd cockroach --home-dir=/opt/cockroachdb
```

Change permissions work dir and extern dir cockroachdb on cockroachdb4-6
```
chown -R cockroach:cockroach /opt/cockroachdb
chown -R cockroach:cockroach /mnt/cockroachdb
chown cockroach /usr/local/bin/cockroach
```

Create systemd service on cockroachdb-4
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-westsumatra1,zone=ind-westsumatra-1b --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

Create systemd service on cockroachdb-5
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-eastjava1,zone=ind-eastjava-1b --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

Create systemd service on cockroachdb-6
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --locality=region=ind-westjava1,zone=ind-westjava-1b --external-io-dir=/mnt/cockroachdb --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
TimeoutStopSec=300
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cockroach
User=cockroach

[Install]
WantedBy=default.target
```

Start systemd on cockroachdb4-6
```
sudo daemon-reload
sudo systemctl start cockroachdb
sudo systemctl enable cockroachdb
sudo systemctl status cockroachdb
```

Check status cluster node on cockroachdb-1
```
cockroach node status --certs-dir=certs --host=$ip_address
```

## Testing Tolerance node on Multi Region Deployment

### Testing tolerance base zone
Create database & table 
```
CREATE DATABASE idn PRIMARY REGION "ind-westsumatra1" REGIONS "ind-westsumatra1";
use idn;
CREATE TABLE peserta (
id INT PRIMARY KEY,
username VARCHAR(50) NOT NULL UNIQUE,
email VARCHAR(100) NOT NULL UNIQUE,
password VARCHAR(255) NOT NULL,
full_name VARCHAR(100),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

check database and check regions
```
SHOW DATABASES;

SHOW REGION FROM DATABASE idn;

SHOW ZONE CONFIGURATION FOR DATABASE idn;
```

testing shutdown salah satu node pada region ind-westsumatra1

### Testing tolerance base on regions

Alter database for multiple regions
```
ALTER DATABASE idn ADD REGION ind-westjava1;
ALTER DATABASE idn SURVIVE ZONE FAILURE;
```

check database and check regions
```
SHOW DATABASES;

SHOW REGION FROM DATABASE idn;

SHOW ZONE CONFIGURATION FOR DATABASE idn;
```

testing shutdown semua node pada salah satu region
