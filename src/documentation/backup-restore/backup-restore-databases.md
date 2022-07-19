# Backup and Restore of Databases

You can securely access your database service using the [VPN](/documentation/cli/vpn.html) built into our [zCLI](/documentation/cli/installation-authorization.html). You can then use any of the tools provided by the database vendor or third-party tools ([MariaDB](/documentation/services/databases/mariadb.html#how-to-backup-restore-database-data), [PostgreSQL](/documentation/services/databases/postgresql.html#how-to-backup-restore-database-data), or [KeyDB](/documentation/services/databases/keydb.html#how-to-backup-restore-database-data)).

This can be used to dump data for use in a locally running database or when migrating away from Zerops. You can also utilize it to initially restore data to Zerops.

Otherwise, Zerops uses [snapshot backup and restore](/documentation/backup-restore/snapshot-backup.html) to safely back up your data.

Full documentation about dumping and restoring data can be found in the [services/databases](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) sections of this documentation.

::: tip USE VPN TO USE ZEROPS DB SERVICE AS DEV DATABASE
Running databases locally, especially if you need to simulate [high availability mode](/documentation/ha/why-should-i-want-high-availability.html), running on multiple containers, is hard. Create a clone of your project, and use the Zerops [VPN](/documentation/cli/vpn.html) to connect to the project and use the database locally.
:::
