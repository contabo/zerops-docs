# MariaDB (MySQL)

Zerops provides a fully managed and scaled MariaDB (MySQL) database service, suitable for both development and production projects using any load. You can choose any option you wish with the knowledge that it will work.

<!-- markdownlint-disable DOCSMD004 -->
::: details Compatibility & Differences to MySQL
Information on the compatibility of MariaDB software with similar, competing software, as well as their differences, can be found [here](https://mariadb.com/kb/en/compatibility-differences).
:::
<!-- markdownlint-enable DOCSMD004 -->

[[toc]]

## Adding the MariaDB Service in Zerops

The Zerops MariaDB service is based on a [Linux LXD container](/documentation/overview/projects-and-services-structure.html#services-containers) with **Ubuntu** **==v22.04.01==**.

There are two possible ways of creating a new MariaDB service. Either manually in the [Zerops GUI](#through-the-zerops-gui-interface), or using the Zerops [import functionality](/documentation/export-import/project-service-export-import.html#how-to-export-import-a-project).

### Using the import functionality

Zerops uses a YAML definition format to describe the structure. View the complete specification of the [import/export syntax in the YAML format](/documentation/export-import/project-service-export-import.html#used-yaml-specification).

To import a MariaDB service, you can use something similar to the following:

```yaml
services:
  # Service will be accessible through zCLI VPN under <protocol>://db:<port>
  - hostname: db
    # Type and version of service used.
    type: mariadb@10.4
    # Whether the service will be run on one or multiple containers.
    # Since this is a simple example, using only one container is fine.
    mode: NON_HA
```

### Through the Zerops GUI interface

#### Which version to choose

You can currently only choose MariaDB version **v10.4** (version 10.4.24, to be precise).

Used as the export & import type: ==`mariadb@10.4`== .

<!-- markdownlint-disable DOCSMD004 -->
::: tip Changing version
Switching must be done manually by creating a new service with another version and migrating data using a [backup / restore](#how-to-backup-or-restore-database-data) pattern.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### Hostname and port

Choose a short and descriptive URL-friendly name, for example, **db**. The following rules apply:

* maximum length **==25==** characters,
* only lowercase ASCII letters **==a-z==** and numbers **==0-9==**,
* **==has to be unique==** in relation to other existing project hostnames,
* the hostname **==can't be changed==** later.

The port will automatically be set to the value of **==3306==** and can't be changed.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Hostname is also used as the default admin user name
The chosen **hostname** is automatically used to create an [admin user account](#default-mariadb-user-and-password) with all privileges and grant options for accessing the database. You can change it later if you prefer.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### HA / non-HA database mode

When creating a new service, you can choose whether the database should be run in **HA** (High Availability) mode, using 3 containers, or **non-HA mode**, using only 1 container. ==**The chosen database mode can't be changed later.**== If you would like to learn more about the technical details and how this service is built internally, take a look at the [MariaDB Service in HA Mode, a deep-dive view](/documentation/overview/how-zerops-works-inside/mariadb-galera-cluster-internally.html).

##### MariaDB in non-HA mode

* great for local development to save money,
* doesn’t require any changes to the existing code,
* not necessary to respect HA mode [specifics](#what-to-bear-in-mind-when-using-HA-mode), but see the recommendation tip below,
* data is stored only in a single container, higher risk of data loss,
* all data changes since the last backup are not recoverable,
* not recommended for production projects.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Recommendation
Even when using the non-HA mode for a production project, we nonetheless recommend you respect all of the [HA mode specifics](#what-to-bear-in-mind-when-using-HA-mode) because you never know when you'll need to switch to HA mode. This is also true for the used storage engine, as InnoDB is the only option in HA mode.
:::
<!-- markdownlint-enable DOCSMD004 -->

##### MariaDB in HA mode

* will run on three containers as a [Galera cluster](https://mariadb.com/kb/en/galera-cluster), each on a **different physical machine**,
* therefore the data is stored redundantly in three places, with no risk of data loss,
* when one container fails, it's automatically replaced with a new one,
* come with two load balancers ([MaxScale](https://mariadb.com/kb/en/maxscale)) in [readwritesplit](https://mariadb.com/kb/en/mariadb-maxscale-25-readwritesplit) mode (no additional cost),
* [asynchronous behavior](#asynchronous-behavior) of a Galera HA cluster,
* only InnoDB storage engine is supported,
* the need to respect all of the [specifics](#what-to-bear-in-mind-when-using-HA-mode) related to a Galera HA cluster,
* recommended for production projects.
  
## How to access a MariaDB database

<!-- markdownlint-disable DOCSMD004 -->
::: warning Don't use additional security protocols for internal communication
The database service is not configured to support direct access using SSL/TLS or SSH protocols for internal communication inside a Zerops project private secured network. This is also the case for access using the Zerops [zCLI](/documentation/cli/installation.html) through a secure VPN channel.
:::
<!-- markdownlint-enable DOCSMD004 -->

### From other services inside the project

Other services can access the database using its **hostname** and **port** environment variables, as they are part of the same private project network. It’s highly recommended to utilize the **==connectionString==** environment variable that Zerops creates automatically for each database, especially when using the HA mode, as it makes sure to include all the information required for HA. Find out more in the dedicated [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) section, related to **connectionString**. See also the list of all automatically generated [environment variables](/documentation/environment-variables/helper-variables.html#mariadb) for the MariaDB service.

For more flexibility with future potential hostname changes, it's always recommended to use them indirectly via [custom environment variables](/knowledge-base/best-practices/how-to-use-environment-variables-efficiently.html) (referencing implicit Zerops environment [variables](/documentation/environment-variables/helper-variables.html#mariadb)) in each project service separately. This allows you to eliminate all direct dependencies in the application code, which in turn means simplicity and increased flexibility. Another reason not to hard-code the values inside your applications is that it can be dangerous because it is easy to commit them (like your credentials) into a repository, potentially exposing them to more people than intended.

### From other Zerops projects

Zerops always sets up a [private dedicated network](/documentation/overview/projects-and-services-structure.html#project) for each project. From this point of view, cross project communication can be done precisely in the same ways described in the following section: [From your public domains (common Internet environment)](#from-your-public-domains-common-internet-environment). There isn't any other specific way. These projects are not directly interconnected.

### From your local environment

The local environment offers ==**not only options for local development**== but also a general ability to ==**manage all Zerops development or production services**== , using zCLI VPN.

To connect to the database from your local workspace, you can utilize the [VPN](/documentation/cli/vpn.html) functionality of our [Zerops zCLI](/documentation/cli/installation.html), as previously mentioned. This allows you to access the database the same way other services inside the project can, but unlike those services, you cannot use references to the environment variables. Therefore, if you need to use them, you should copy the values manually through the **How To Access** / **Database access details** section of the service detail in your application and use them in your private local configuration strategy.

![MariaDB Service](./images/MariaDB-Database-Access-Details.png "Database Access Details")

### From your public domains (common Internet environment)

You can't access the MariaDB service directly in any way. You have to use one of the runtime environment services ([Node.js](/documentation/services/runtimes/nodejs.html), [Golang](/documentation/services/runtimes/golang.html), or [PHP](/documentation/services/runtimes/php.html)) and go indirectly through them in a programmatic way. They should implement their authentication logic to access the MariaDB service.

To understand this better, take a look at the following section: [With external access](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html#with-external-access) of **Typical schemas of Zerops Projects**.

## Default MariaDB user and password

Zerops automatically creates a user with all privileges and grant options when creating the service, where the name of **==user==** is based on the selected **hostname**, and the **==password==** is randomly generated. These are saved to the environment variables **user** and **password** and can be referenced from other services the same way as with a **connectionString**.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Zerops doesn’t keep both places in sync
If you change your password inside the MariaDB database directly, the change is not reflected in the environment variable and vice versa. It’s up to you to keep these up to date through the **Service env. Variables** section of the service detail in your application.

![MariaDB Service](./images/MariaDB-Database-Access-Change-Password.png "Database Access Change Password")
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: tip Default Zerops maintenance user
For system maintenance reasons, the `zps` user is also automatically created with all privileges. It's important not to change it in any way. Otherwise, there is a risk of disrupting the correct functionality, especially in HA mode.
:::
<!-- markdownlint-enable DOCSMD004 -->

## Default MariaDB database

A new database with the name based on the selected **hostname** is created during the initial service setup (for example, database **db** if the chosen hostname was **db**). This database is created with the default `utf8mb4` charset, allowing to save 4-Bytes characters, like Emoji, without any problem. MariaDB generally doesn't require to specify a database when a new connection is created, and it's possible to connect just to the database server. But it's usual to connect using **connectionString** and add the target database as the `/<hostname>`.

## Default hardware configuration and autoscaling

* Each MariaDB container (1 in non-HA, 3 in HA) starts with 1 vCPU, 1 GB RAM, and 1 GB of disk space.
* Zerops will only automatically scale the database [vertically](/documentation/automatic-scaling/how-automatic-scaling-works.html#vertical-scaling) (both in non-HA and HA mode).
* The [horizontal autoscaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) in HA mode is not applied because of optimal performance.

## How to backup or restore database data

### Using your favorite database management tool

Install any of your favorite database administration tools locally. For example, you can use [DataGrip](https://www.jetbrains.com/datagrip), [DbVisualizer](https://www.dbvis.com), [MySQL Workbench](https://dev.mysql.com/downloads/workbench) multi-platform database administration tools, or [Sequel Pro](https://www.sequelpro.com), [Querious](https://www.araelium.com/querious), specifically on the Mac platform. You can find the list with many others on the [Graphical and Enhanced Clients](https://mariadb.com/kb/en/graphical-and-enhanced-clients) page.

First, connect to your Zerops project using [zCLI](/documentation/cli/installation.html) & [VPN](/documentation/cli/vpn.html) and then you can use ==`db:3306`== as the endpoint. After that, connect to the database service from your installed database management tool, as in the example below with the **Querious**:

![Querious](./images/Querious-Connect.png "Querious Connect Dialog")

<!-- markdownlint-disable DOCSMD004 -->
::: tip Connection security settings
As you are using a secure VPN channel already, and the database service is located on the internal Zerops project private secured network, you don't need to apply any additional security layers such as SSH or SSL/TLS. For this reason, the database service is not configured to support access using SSL/TLS or SSH protocols for internal communication inside a Zerops project. Find out more about how the Zerops project works with [external access](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html#with-external-access).
:::
<!-- markdownlint-enable DOCSMD004 -->

Now you can easily use the export/import built-in functions to backup/restore database data to/from your local file system.

![Querious](./images/Querious-Export.png "Querious Export")

### Using mariadb/mysql CLI

Again, first access your Zerops project using [zCLI](/documentation/cli/installation.html) & [VPN](/documentation/cli/vpn.html). The `mariadb`/`mysql` CLI client need to be already installed locally. It comes with each local installation of a MariaDB server either on the [Mac platform with Homebrew](https://mariadb.com/resources/blog/installing-mariadb-10-1-16-on-mac-os-x-with-homebrew) or [Linux](https://mariadb.com/downloads). You can also use a multi-platform [MySQL Shell](https://dev.mysql.com/downloads/shell), `mysql-client` on [Mac](https://formulae.brew.sh/formula/mysql-client), or `mysql-client` / `mysql` on Linux (where concrete steps depend on each distribution version). The client CLI is a part of [MySQL Workbench](https://dev.mysql.com/downloads/workbench) as well.

#### Logical database backup

```bash
mysqldump -h [hostname] -u [user] -p[password] -x [database] > [filename].dump
```

And when used values:

* hostname = **==db==** (specified when MariaDB Service was created)
* user = **==db==** (automatically created with the same name as the hostname)
* password = **==xxxxxxxxxxxxxxxx==** (automatically created as a random value)
* database = **==main==** (created by a user later as a part of the project)
* filename = **==main==** (specified filename to store the database backup)

```bash
mysqldump -h db -u db -pxxxxxxxxxxxxxxxx -x main > main.dump
```

#### Logical database restore

```bash
mysql -h [hostname] -u [user] -p[password] [database] < [filename].dump
```

<!-- markdownlint-disable DOCSMD004 -->
::: tip Available aliases in the command-line client
Starting with MariaDB 10.4.6, `mariadb` is a [symlink](https://mariadb.com/kb/en/mariadb-command-line-client) to `mysql`. The same is true for `mariadb-dump` as a [symlink](https://mariadb.com/kb/en/mariadb-dump) to `mysqldump`.
:::
<!-- markdownlint-enable DOCSMD004 -->

### Using Adminer / phpMyAdmin

Create a new Zerops [PHP service](/documentation/services/runtimes.html#php) and [deploy](/documentation/services/runtimes/php.html#how-to-deploy-application-code) a pre-prepared [Adminer](/knowledge-base/how-to-do/how-to-prepare-adminer-application-kit.html) or [phpMyAdmin](/knowledge-base/how-to-do/how-to-prepare-phpmyadmin-application-kit.html) application kit. After enabling a [Zerops subdomain](/documentation/routing/zerops-subdomain.html) on such a service, you can access the application through its root URL. You can access the MariaDB database service using its hostname, port, user, and password. After that, you can use its built-in export/import functions to backup/restore database data to/from your local file system.

![Adminer](./images/Adminer-Login.png "Adminer Login")

You can also use the Zerops [import functionality](/documentation/export-import/project-service-export-import.html) to quickly add a service with **Adminer** to your project. Use the Zerops [recipe-adminer](https://github.com/zeropsio/recipe-adminer) import.

```yaml
services:
  # Service will be accessible through zCLI VPN under: http://adminer
- hostname: adminer
  # Type and version of service used.
  type: php-apache@8.0+2.4
  # Whether the service will be run on one or multiple containers.
  # Since this is a utility service, using only one container is fine.
  minContainers: 1
  maxContainers: 1
  # Folder name used as the root of the publicly accessible web server content.
  documentRoot: public
  # Repository that contains adminer code with build and deploy instructions.
  buildFromGit: https://github.com/zeropsio/recipe-adminer@main
```

After that you can use **Adminer** either using [Zerops VPN](/documentation/cli/vpn.html) built into the [zCLI](/documentation/cli/installation.html) through URL `http://<hostname>:<port>` (here, it means: `http://adminer`) or enable Zerops [subdomain access](/documentation/routing/zerops-subdomain.html).

<!-- markdownlint-disable DOCSMD004 -->
::: tip Explanation of the phpMyAdmin security warning
Zerops Routing Service (see the schema of a Zerops project with [external access](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html#with-external-access)) takes care of SSL certificate management and internal translation of HTTPS protocol to HTTP for all project services. This is why **phpMyAdmin** can see the difference between using HTTPS protocol on a client-side and HTTP protocol on a server-side. **From a Zerops point of view, it's not a security risk**. It's the only positive falsy notification from the phpMyAdmin side (not supporting the `HTTP_X_FORWARDED_PROTO` HTTP header that says it has happened).
:::
<!-- markdownlint-enable DOCSMD004 -->

![phpMyAdmin](./images/phpMyAdmin-Login.png "phpMyAdmin Login")

## What to bear in mind when using HA mode

### Asynchronous behavior

<!-- markdownlint-disable DOCSMD004 -->
::: warning Be sure you understand correctly
When data is stored in a MariaDB cluster (always through its current primary database instance), it is replicated across other standby replica instances asynchronously. This means that if one SQL statement stores some data, the following immediate select query may not retrieve the same data. The reason is that the given statement will be executed against another replica instance. If required to get the same data, it's necessary to encapsulate both commands into a single SQL transaction, which will always be executed against the primary instance (because of the write operation that is included). The same case would be with two immediately following SELECT statements to get the same data. Again, encapsulating of both commands into a single SQL transaction guarantees their execution against a selected but the same replica instance.

You can also force synchronization to wait for causality checks on a cluster by [wsrep_sync_wait](https://mariadb.com/docs/reference/mdb/system-variables/wsrep_sync_wait) bitmask. Enabling it ensures that certain types of queries always execute against the most up to date database state, at the expense of query performance. [Sample usage](https://mariadb.com/kb/en/galera-cluster-system-variables/#wsrep_sync_wait) for a critical read that must have the most up-to-date data:

```sql
 SET SESSION wsrep_sync_wait=1;
 SELECT ...;
 SET SESSION wsrep_sync_wait=0;
```

:::
<!-- markdownlint-enable DOCSMD004 -->

### Non-database local data

Each container has separate local disk space, which can theoretically be used by appropriate APIs of the database service and thus store data outside the replicated contents of the database. It should be noted that such data is only reserved for this particular instance, not mirrored across the MariaDB Galera cluster, nor is it backed up. It will not be migrated if such a container is deleted due to failure. Also, separate direct access to an individual MariaDB instance is not supported in any way.

We don't recommend using any functionality of [SELECT INTO OUTFILE](https://mariadb.com/kb/en/select-into-outfile), [SELECT INTO DUMPFILE](https://mariadb.com/kb/en/select-into-dumpfile), [LOAD_FILE](https://mariadb.com/kb/en/load_file), or [LOAD DATA](https://dev.mysql.com/doc/refman/8.0/en/load-data.html). You can't save/load such data directly to/from any shared storage, and in the Galera HA cluster environment, it is impossible to predict which container the selected data will be stored on. Instead, use the the [export/import](/documentation/services/databases/mariadb.html#how-to-backup-or-restore-database-data) mechanism's standard functionality.

### Selected specifics of a Galera HA cluster

* Only InnoDB storage engine is supported.
* No support for explicit locks, including LOCK TABLES, FLUSH TABLES {explicit table list} WITH READ LOCK, GET_LOCK, RELEASE_LOCK, etc. is provided. **These problems can be avoided by using transactions.** Global locking operators like FLUSH TABLES WITH READ LOCK are supported.
* All database tables should have a primary key (multi-column primary keys can also be used). **DELETE operations are unsupported on tables without a primary key.**
* Distributed [XA transactions](https://mariadb.com/kb/en/xa-transactions) are not supported.
* [AUTO_INCREMENT](https://galeracluster.com/library/kb/auto-increment-multiples.html) automatically uses the number of nodes in a Galera cluster as an increment. This means that it's not possible to rely on the next auto-generated value being max(x)+1.
* Other details about the limitations and behavior of MariaDB Galera clusters and MaxScale load balancers are available at [MariaDB Galera Cluster - Known Limitations](https://mariadb.com/kb/en/mariadb-galera-cluster-known-limitations) and [Limitations and Known Issues within MariaDB MaxScale](https://mariadb.com/kb/en/mariadb-maxscale-25-limitations-and-known-issues-within-mariadb-maxscale).
