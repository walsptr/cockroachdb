# CockroachDB Administration

## User tools - Command Line Interface
Access SQL CLI
```
cockroach sql --certs-dir=certs --host=$ip_address
```

## Manage Database
Show databases on cluster
```
SHOW DATABASES;
```

Create a new database
```
CREATE DATABASE training;
```

Using database
```
USE training;
```

Delete database
```
DROP training;
```

## Manage tables
Use database
```
USE training;
```

create table
```
CREATE TABLE peserta (
id INT PRIMARY KEY,
username VARCHAR(50) NOT NULL UNIQUE,
email VARCHAR(100) NOT NULL UNIQUE,
password VARCHAR(255) NOT NULL,
full_name VARCHAR(100),
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

show columns table
```
SHOW COLUMNS FROM peserta;
```

Insert data to table
```
INSERT INTO peserta (id, username, email, password, full_name)
VALUES
(1, 'andi123', 'andi@example.com', 'password123', 'Andi Saputra'),
(2, 'budi456', 'budi@example.com', 'securepass456', 'Budi Santoso'),
(3, 'citra789', 'citra@example.com', 'rahasia789', 'Citra Lestari'),
(4, 'dian321', 'dian@example.com', 'passdian321', 'Dian Pratama'),
(5, 'eka654', 'eka@example.com', 'ekapass654', 'Eka Putri');
```

select table
```
SELECT * FROM peserta;
```

## Manage user & Grant access

showing user
```
SHOW USERS;
```

create user
```
CREATE USER andi WITH LOGIN PASSWORD 'testing123';
```

testing login with user
```
cockroach sql --certs-dir=certs --host=$ip_address --user=andi
```
lakukan test seperti masuk ke database training dan select table peserta (harusnya tidak bisa select table tersebut), testing juga untuk create table sendiri;

grant table to user
```
GRANT ALL ON TABLE peserta TO andi;
```
testing select lagi dan coba insert

grant database
```
CREATE DATABASE testing;

use testing;

CREATE TABLE testable (
id INT PRIMARY KEY
);

GRANT ALL ON DATABASE testing TO andi;
```
lakukan testing seperti select table, create table dll

> [!NOTE]
> by default user bisa akses semua database dan create table didalamnya, namun tidak bisa melakukan operation pada table yang bukan miliknya
> by default role tidak bisa login, berbeda dengan user yang defaultnya bisa login
> reference grant database: https://www.cockroachlabs.com/docs/v24.3/grant#grant-privileges-on-all-tables-in-a-database-or-schema

## Licensing
- Create account on https://www.cockroachlabs.com/
- Go to organization --> Enterprise License menu
- Click Create license

Check enterprise license
```
SHOW CLUSTER SETTING enterprise.license;
```

Add license to cluster
```
SET CLUSTER SETTING enterprise.license = 'crl-0-xxxxx';
```
