---
title: "Restoring a malformed Grafana's database"
date: 2019-11-08T23:33:47+01:00
draft: false
---

# Restoring a malformed Grafana's database

```
journalctl -u grafana-server.service
> (...) database disk image is malformed
```

Thanks to [Jason L. Froebe](https://www.froebe.net/blog/2015/05/27/error-sqlite-database-is-malformed-solved/):

To check if indeed is corrupted:

```
# sqlite3 /var/lib/grafana/grafana.db
sqlite> pragma integrity_check;
Error: database disk image is malformed
```

Export the schema and data:
```
sqlite> .mode insert
sqlite> .output mydb_export.sql
sqlite> .dump
sqlite> .exit
```

Backup the original database:
```
# mv /var/lib/grafana/grafana.db /var/lib/grafana/grafana.db.backup
```

Create a new database and import:
```
# sqlite3 /var/lib/grafana/grafana.db < mydb_export.sql
```

Update statistics:
```
# sqlite3 /var/lib/grafana/grafana.db
sqlite> analyze;
sqlite> .exit
```

Restore permissions and user:
```
# chmod 664 /var/lib/grafana/grafana.db
# chown grafana /var/lib/grafana/grafana.db
```
