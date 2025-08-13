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

Setup sudo no passwd for user
```
sudo vim /etc/sudoers

$user ALL=(ALL:ALL) NOPASSWD: ALL
```

Setup SSH Config
```
vim ~/.ssh/config

Host cockroachdb-1
  HostName cockroachdb-1
  User $user
  StrictHostKeyChecking no
Host cockroachdb-2
  HostName cockroachdb-2
  User $user
  StrictHostKeyChecking no
Host cockroachdb-3
  HostName cockroachdb-4
  User $user
  StrictHostKeyChecking no
```

Setup SSH Public key
```
ssh-keygen -t rsa
ssh-copy-id cockroachdb-1
ssh-copy-id cockroachdb-2
ssh-copy-id cockroachdb-3
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

Create work dir cockroachdb on all node
```
mkdir -p /opt/cockroachdb/certs
```

Create CA file on cockroachdb-1
```
cd /opt/cockroachdb
cockroach cert create-ca --certs-dir=certs --ca-key=certs/ca.key
```

Copy CA to all node cluster
```
scp ~/certs/ca.crt ~/certs/ca.key cockroachdb-2:/tmp
scp ~/certs/ca.crt ~/certs/ca.key cockroachdb-3:/tmp
```

Move the CA file and key to Work Dir
```
ssh cockroachdb-2
sudo mv /tmp/ca.* /opt/cockroachdb/certs

ssh cockroachdb-3
sudo mv /tmp/ca.* /opt/cockroachdb/certs
```

Create client cert on all node
```
cockroach cert create-client root --certs-dir=certs --ca-key=certs/ca.key
```

Setup server cert on all node
```
cockroach cert create-node localhost $hostname $ip_address $ip_load_balancer $hostname_load_balancer --certs-dir=certs --ca-key=certs/ca.key
```
> [!NOTE]
> untuk bagian $ip_load_balancer dan $hostname_load_balancer itu opsional, tergantung apakah architecture kita menggunakan load balancer atau nggk didepannya

Setup non login user for cockroachdb service on all node
```
useradd cockroach --shell=/sbin/nologin
```

Change permissions work dir cockroachdb on all node
```
chown -R cockroach:cockroach /opt/cockroachdb
chown cockroach /usr/local/bin/cockroach
```

Create systemd service on all node
```
vim /etc/systemd/system/cockroachdb.service

[Unit]
Description=Cockroach Database cluster node
Requires=network.target

[Service]
Type=notify
WorkingDirectory=/opt/cockroachdb
ExecStart=/usr/local/bin/cockroach start --cache=.35 --max-sql-memory=.35 --certs-dir=certs --listen-addr=$cockroachdb --advertise-addr=$cockroachdb --join='cockroachdb-1:26257 ,cockroachdb-2:26257 ,cockroachdb-3:26257'
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
> [!NOTE]
> - parameter cache dan max sql memory merupakan tunning untuk proses caching dan consume memory, by default itu 128MiB dan untuk production bagusnya itu 25%/.25 atau lebih.
> - untuk custom bisa gunakan rumus berikut: (2 * --max-sql-memory) + --cache <= 80% of system RAM, agar terjaga dari OOM events
> - di sarankan juga untuk provisioning RAM 4GB/vCPU dan Storage 320 GiB/vCPU 
> - reference: https://www.cockroachlabs.com/docs/stable/recommended-production-settings#cache-and-sql-memory-size


Start systemd and Init Cluster on cockroachdb-1
```
sudo daemon-reload
sudo systemctl start cockroachdb
sudo systemctl enable cockroachdb
sudo systemctl status cockroachdb

cockroach init --certs-dir=certs --host=$ip_address

cockroach node status --certs-dir=certs --host=$ip_address
ss -tulpn | grep 26257
```

Start systemd on Another Node
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

## cockroach start commands information
```
--certs-dir = letak directory yang berisi file crt
--listen-addr = listen ip address
--advertise-addr = ip address yang akan di advertise
--join = ip" node yang join cluster
--store = untuk custom store data
```
