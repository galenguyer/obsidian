# Setting up Replication with PostgreSQL

Create a user for replication on the primary server:
```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'secret';
```

Add the following line to the end of `/var/lib/pgsql/12/data/pg_hba.conf` on the master.
```txt
host    replication     replicator      [STANDBY_IP]/32     md5
```

Restart postgres to load this change.
```bash
systemctl restart postgresql-12.service
```

On the standby server, make sure `/var/lib/pgsql/12/data/` is emtpy and run the following:
```bash
pg_basebackup -h [PRIMARY_IP] -D /var/lib/pgsql/12/data -U replicator -P -v -R -X stream -C -S [REPLICA_NAME]
```

-   `-h` – specifies the host which is the master server.
-   `-D` – specifies the data directory.
-   `-U` – specifies the connection user.
-   `-P` – enables progress reporting.
-   `-v` – enables verbose mode.
-   `-R` – enables the creation of recovery configuration: Creates a **standby.signal** file and append connection settings to **postgresql.auto.conf** under the data directory.
-   `-X` – used to include the required write-ahead log files (WAL files) in the backup. A value of stream means to stream the WAL while the backup is created.
-   `-C` – enables the creation of a replication slot named by the -S option before starting the backup.
-   `-S` – specifies the replication slot name.

Once this is done, start postgres on the replica.
```bash
systemctl start postgresql-12
```