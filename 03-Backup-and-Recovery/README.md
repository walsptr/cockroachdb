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

## Import
## Manual Backup
## Scheduling Backup
## Restore
