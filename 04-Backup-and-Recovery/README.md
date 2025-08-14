# CockroachDB Backup & Recovery

## Export
export tables data
```
EXPORT INTO CSV 'nodelocal://1/csv' FROM TABLE training.peserta;
```

export data using changefeed
```
CREATE CHANGEFEED FOR TABLE training.peserta INTO 'nodelocal://1/csv' WITH initial_scan = 'only', format=csv;
```
> [!NOTE]
> yang membedakan export biasa dan changefeed merupakan cara kerjanya, di changefeed kita bisa pause and resume proses job nya. 

Backup dengan SQL Client
```
cockroach sql --host=$ip_address --certs-dir=/opt/cockroachdb/certs -e "SELECT * from training.peserta" --format=csv > /mnt/cockroachdb/peserta.csv
```

## Import

### Import CSV File

create new database for testing import csv file
```
create database kelas;
```

create new table
```
CREATE TABLE siswa (
id INT PRIMARY KEY,
username VARCHAR(50) NOT NULL UNIQUE,
email VARCHAR(100) NOT NULL UNIQUE,
password VARCHAR(255) NOT NULL,
full_name VARCHAR(100),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Import data from peserta.csv
```
IMPORT INTO siswa (id, username, email, password, full_name, created_at) CSV DATA ('nodelocal://1/peserta.csv') WITH SKIP = '1';
```

### Import from PGDUMP

create new database for testing import from pgdump file
```
create database class;
```

Import the sql file
```
IMPORT TABLE peserta FROM PGDUMP 'nodelocal://1/peserta_dump.sql' WITH ignore_unsupported_statements;
```

## Manual Backup
Backup table
```
BACKUP peserta.training INTO 'nodelocal://1/manual-backup-table';
```

Backup Database
```
BACKUP DATABASE peserta INTO 'nodelocal://1/manual-backup-database';
```

Backup All Cluster
```
BACKUP INTO 'nodelocal://1/manual-backup-cluster';
```

## Scheduling Backup
Full Backup cluster
```
CREATE SCHEDULE full_backup_cluster FOR BACKUP INTO 'nodelocal://1/fullbackup-cluster' RECURRING '@daily' FULL BACKUP ALWAYS WITH SCHEDULE OPTIONS first_run = 'now';
```

Incremental backup cluster
```
CREATE SCHEDULE incremental_backup FOR BACKUP INTO 'nodelocal://1/increment-backup' WITH revision_history RECURRING '@daily' WITH SCHEDULE OPTIONS first_run = 'now';
```

Incremental backup database
```
CREATE SCHEDULE schedule_label FOR BACKUP DATABASE training INTO 'nodelocal://1/increm-backup-db' WITH revision_history RECURRING '@daily' WITH SCHEDULE OPTIONS first_run = 'now';
```

## Restore
Restore full cluster from latest backup
```
RESTORE FROM LATEST IN 'nodelocal://1/fullbackup-cluster';
```

Restore specific time backup
```
RESTORE FROM '2023/03/23-213101.37' IN 'nodelocal://1/fullbackup-cluster';
```

Restore database from latest
```
RESTORE DATABASE training FROM LATEST IN 'nodelocal://1/fullbackup-cluster';
```

Restore table from latest
```
RESTORE TABLE training.peserta FROM LATEST IN 'nodelocal://1/fullbackup-cluster';
```

Restore table to another database
```
RESTORE TABLE training.peserta FROM LATEST IN 'nodelocal://1/fullbackup-cluster' WITH into_db = 'newtraining';
```

Restore database with different name
```
RESTORE DATABASE training FROM LATEST IN 'nodelocal://1/fullbackup-cluster' WITH new_db_name = 'trainingnew';
```
