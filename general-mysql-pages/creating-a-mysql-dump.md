# Creating a MySql dump

## Copying an existing schema to a new database:

Tables only as in only table structure with no data:

```
mysqldump --login-path=root  --no-data --routines --skip-triggers --set-gtid-purged=OFF TABLE_NAME > TABLE_NAME_tables.sql
```

{% hint style="info" %}
Doing the table structure first prevents locks occurring on the DB and is more reliable.
{% endhint %}

Next we want to dump the table data to do this we do the following for example using control and teti

#### Control table

```bash
mysqldump --login-path=root --no-data --routines --skip-triggers --set-gtid-purged=OFF -v forcelink_control > forcelink_control_tables.sql
mysqldump --login-path=root --routines --skip-triggers --set-gtid-purged=OFF --ignore-table=forcelink_control.processlist_history -v forcelink_control > forcelink_control_data.sql
```

#### All other tables

```
mysqldump --login-path=root --no-data --routines --skip-triggers --set-gtid-purged=OFF -v forcelink_teti > forcelink_teti_tables.sql
mysqldump --login-path=root --routines --skip-triggers --set-gtid-purged=OFF --ignore-table=forcelink_teti.audit_trail -v forcelink_teti > forcelink_teti_data.sql
```
