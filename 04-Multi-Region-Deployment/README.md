# Multi Region Deployment CockroachDB

## Setup cockroachdb1-3
Edit systemd file on cockroachdb1
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

Edit systemd file on cockroachdb2
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

Edit systemd file on cockroachdb3
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

## Setup Cockroachdb4-6
lakukan langkah-langkah seperti pada setup cockroachdb di awal untuk cockroachdb4-6
