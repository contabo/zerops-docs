# Export & import of Zerops projects and services structure

Zerops export and import functionality can be used to make a copy of an existing project/service or as an alternative to adding a new project/service manually through the Zerops GUI. It is important to note that only structure, configuration, and environment variables are exported/imported. Restoring application data related to databases ([PostgreSQL](/documentation/services/databases/postgresql.html#how-to-backup-restore-database-data), [MariaDB](/documentation/services/databases/mariadb.html#how-to-backup-restore-database-data), [KeyDB](/documentation/services/databases/keydb.html#how-to-backup-restore-database-data)), storage ([Object Storage](/documentation/services/storage/s3.html#using-rclone-as-a-local-management-tool), Shared Storage), and engines ([RabbitMQ](/documentation/services/message-brokers/rabbitmq.html#how-to-backup-restore-queues-data)), deploying applications, and setting up [public routing](/documentation/routing/using-your-domain.html) is up to the specific user.

[[TOC]]

## How to export / import a project

There are several places in the Zerops GUI where you can do this. The primary one for export is the main project menu, as shown in the image below. The project will be exported along with all of its services.

![Zerops exports](./images/Project-Export.png "Zerops project export")

You can import the entire project, including all of its services, on the same page you use to manually create a new project.

![Zerops imports](./images/Project-Import.png "Zerops project import")

## How to export / import a service

The place to export a project service is its service menu, as shown below.

![Zerops exports](./images/Service-Export.png "Zerops service export")

Importing a project service can be done in the project menu, as shown in the [image at the top](#how-to-export-import-a-project).

## YAML specification

Zerops uses a YAML definition format to describe the structures. If you carry out a project export, you'll get something similar to the following. You can see two main parts, `project` and `services`. If you export a service separately, only the `services` part will be shown relative to the exported service.

```yaml
project:
  name: Example
  tags:
  - DEMO
  - ZEROPS
services:
- hostname: db
  type: mariadb@10.4
  priority: 2
  mode: HA
- hostname: app
  type: nodejs@14
  priority: 1
  minContainers: 2
  verticalAutoscaling:
    minCpu: 5
    maxCpu: 10
  ports:
  - port: 3000
    httpSupport: true
  - port: 9090
    protocol: UDP
    httpSupport: false
  envVariables:
    jwt_token_secret: M3rW31Ne%T@bRk
    CONNECTION_STRING: ${db_connectionString}
- hostname: sharedstorage
  type: shared-storage
  mode: NON_HA
  verticalAutoscaling:
    minDisk: 25.5
```

### project

**`project`**

A project structure uses `name`, `description`, and `tags` items.

#### name

`name`: string

A project name. When exported, it's pre-pended by the prefix of `copy of` to differentiate it.

#### description

`description`: string (optional)

A project description, if entered.

#### tags

`tags`: Array[string] (optional)

A sequence of project tags, if entered. Each on a separate line with the opening dash.

### services

**`services`**

A sequence of project services (1~N). Several general items are shared across all service structures, exceptional ones are used only in some cases.

#### hostname

`hostname`: string

A chosen short and descriptive, URL-friendly unique service name. Related to [PostgreSQL](/documentation/services/databases/postgresql.html#hostname-and-ports), [MariaDB](/documentation/services/databases/mariadb.html#hostname-and-port), [KeyDB](/documentation/services/databases/keydb.html#hostname-and-port), [Node.js](/documentation/services/runtimes/nodejs.html#port), [Golang](/documentation/services/runtimes/golang.html#port), [PHP](/documentation/services/runtimes/php.html#hostname-and-port), [RabbitMQ](/documentation/services/message-brokers/rabbitmq.html#hostname-and-ports), [Object Storage](/documentation/services/storage/s3.html#object-storage-name), and [Shared Storage](/documentation/services/storage/shared.html#shared-storage-name).

#### type

`type`: dictionary

A service type and its chosen version. Each of the following service documentation specifies the options available: [PostgreSQL](/documentation/services/databases/postgresql.html#which-version-to-choose), [MariaDB](/documentation/services/databases/mariadb.html#which-version-to-choose), [KeyDB](/documentation/services/databases/keydb.html#which-version-to-choose), [Node.js](/documentation/services/runtimes/nodejs.html#which-version-to-choose), [Golang](/documentation/services/runtimes/golang.html#which-version-to-choose), [PHP](/documentation/services/runtimes/php.html#which-version-to-choose), [RabbitMQ](/documentation/services/message-brokers/rabbitmq.html#which-version-to-choose), [Object Storage](/documentation/services/storage/s3.html#which-version-to-choose), and [Shared Storage](/documentation/services/storage/shared.html#which-version-to-choose).

Comprehensive table of available types:

|Service        |Types                                                      |
|:--------------|:----------------------------------------------------------|
|PostgreSQL     |postgresql@12                                              |
|MariaDB        |mariadb@10.4                                               |
|KeyDB          |keydb@6                                                    |
|Node.js        |nodejs@16, nodejs@14, nodejs@12                            |
|Golang         |golang@1                                                   |
|PHP+Apache     |php-apache@8.1+2.4, php-apache@8.0+2.4, php-apache@7.4+2.4 |
|PHP+Nginx      |php-nginx@8.1+1.20, php-nginx@8.0+1.20, php-nginx@7.4+1.20 |
|Static server  |nginx@1.20                                                 |
|RabbitMQ       |rabbitmq@3                                                 |
|Object storage |object-storage                                             |
|Shared storage |shared-storage                                             |

#### priority

`priority`: integer [>= 1] (optional)

This allows you to control precisely the order in which the services will be created when the import YML definition is processed. There are situations, for example, when a database service (as MariaDB or PostgreSQL) has to already be running before a new application build phase of a runtime (Node.js, Golang, PHP) is started. The following rules are applied:

* services without the explicit `priority` option are grouped (`priority: 0`) and created last,
* services with the explicit `priority` option are ordered in an descending manner,
* `priority` values may not be unique and do not have to represent a continuous numerical series,
* lower priority services will not be created until all higher priority ones are available.

If more of the same `priority` values exist (including none priorities), the creation of all services within the same priority group is invoked at the exact moment, and the final availability of these services is not explicitly controlled (depends on the service type and other parameters).

![Zerops import priorities](./images/Timeline_Services_Priorities.png "Zerops import priorities")

#### mode

`mode`: dictionary

Affects whether a service should be run in ==**`HA`**== (High Availability) mode, using 3 or more containers, or ==**`NON_HA`**== mode, using only 1 container. It is related only to [PostgreSQL](/documentation/services/databases/postgresql.html#ha-non-ha-database-mode), [MariaDB](/documentation/services/databases/mariadb.html#ha-non-ha-database-mode), [KeyDB](/documentation/services/databases/keydb.html#ha-non-ha-database-mode), [RabbitMQ](/documentation/services/message-brokers/rabbitmq.html#ha-non-ha-mode), and [Shared Storage](/documentation/services/storage/shared.html#ha-non-ha-shared-storage-mode).

Comprehensive table of available modes:

|Service        |Mode         |
|:--------------|:------------|
|PostgreSQL     |NON_HA or HA |
|MariaDB        |NON_HA or HA |
|KeyDB          |NON_HA or HA |
|RabbitMQ       |NON_HA or HA |
|Shared storage |NON_HA or HA |

<!-- markdownlint-disable DOCSMD004 -->
::: tip Zerops runtime and static server services & High Availability
For runtime services ([Node.js](/documentation/services/runtimes/nodejs.html#ha-non-ha-runtime-environment-mode), [Golang](/documentation/services/runtimes/golang.html#ha-non-ha-runtime-environment-mode), [PHP](/documentation/services/runtimes/php.html#ha-non-ha-runtime-environment-mode), and [Static server](/documentation/services/static-servers/nginx.html#ha-non-ha-mode) service) there is the [minContainers](#minContainers) property that allows a user to set a service high availability level.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: warning Object Storage Service
The [Object Storage](/documentation/services/storage/s3.html#used-technology) service is the fully Zerops managed service and a user can't influence its functionality by this property.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### minContainers

`minContainers`: integer [>= 1 and <= 4]

Affects how many containers a service should run on as a minimum. Zerops applies automatically the [horizontal scaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) up to the 4 containers as a maximum, running the service in **high availability mode**. It is related only to [Node.js](/documentation/services/runtimes/nodejs.html#ha-non-ha-runtime-environment-mode), [Golang](/documentation/services/runtimes/golang.html#ha-non-ha-runtime-environment-mode), [PHP](/documentation/services/runtimes/php.html#ha-non-ha-runtime-environment-mode), and [Static server](/documentation/services/static-servers/nginx.html#ha-non-ha-mode).

<!-- markdownlint-disable DOCSMD004 -->
::: tip How to set up the non high availability mode
If you want to run the service only on 1 container, you have to set `minContainers: 1` and also [maxContainers](#maxContainers) parameter as `maxContainers: 1`.
:::
<!-- markdownlint-enable DOCSMD004 -->

Comprehensive table of available values:

|Service        |minContainers |
|:--------------|:-------------|
|Node.js        |>= 1 and <= 4 |
|Golang         |>= 1 and <= 4 |
|PHP+Apache     |>= 1 and <= 4 |
|PHP+Nginx      |>= 1 and <= 4 |
|Static server  |>= 1 and <= 4 |

The upper limit for the minimum of containers can be generally different for each service type and can be changed in the future.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Zerops database, message brokers, and shared storage services & High Availability
There is the [mode](#mode) property that allows a user to set whether a service should run in HA mode or not.
:::
<!-- markdownlint-enable DOCSMD004 -->

#### maxContainers

`maxContainers`: integer [>= [minContainers](#minContainers) and <= 4] optional

Affects how many containers a service should run on as a maximum when Zerops applies the [horizontal scaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) feature. It is related only to [Node.js](/documentation/services/runtimes/nodejs.html#ha-non-ha-runtime-environment-mode), [Golang](/documentation/services/runtimes/golang.html#ha-non-ha-runtime-environment-mode), [PHP](/documentation/services/runtimes/php.html#ha-non-ha-runtime-environment-mode), and [Static server](/documentation/services/static-servers/nginx.html#ha-non-ha-mode).

The upper limit for the maximum of containers can be generally different for each service type and can be changed in the future.

#### ports

`ports`: Array[ServicePort] (optional)

A sequence of service ports (1~N). Each one contains `port`, `protocol`, and `httpSupport` items.

Required only for the [Node.js](/documentation/services/runtimes/nodejs.html#port) and [Golang](/documentation/services/runtimes/golang.html#port) runtime environment services, where you can set or change it. **You have to enter at least one port.** The rest of the services have preset ports which can't be changed. That's why this part is not included at the service level, and if entered, is ignored.

##### port

`port`: integer

A chosen port number.

##### protocol

`protocol`: dictionary (optional)

A chosen protocol. The default value is the ==`TCP`== and it does not have to be entered. The other possible option is the ==`UDP`== value.

##### httpSupport

`httpSupport`: boolean (optional)

The default value ==`true`== indicates if a web server runs on the port (HTTP application protocol is supported), otherwise value ==`false`== is used. If enabled, it means that you can even map [public Internet domains](/documentation/routing/using-your-domain.html#using-your-domain-to-access-a-service) with the option of automatic support for SSL certificates (it also works for [Zerops subdomains](/documentation/routing/zerops-subdomain.html#zerops-subdomain-for-previews)).

#### envVariables

`envVariables`: Map[key:value] (optional)

A sequence of [service environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) (0~N).

##### key

`key`: string

An environment variable key.

##### value

`value`: string

An environment variable content.

```yaml
- hostname: app
  type: nodejs@14
  minContainers: 1
  maxContainers: 1
  ports:
  - port: 3000
    httpSupport: true
  envVariables:
    jwt_token_secret: M3rW31Ne%T@bRk
    CONNECTION_STRING: ${db_connectionString}
```

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using modificators for environment variables
Sometimes is helpful to have additional functionality to transform environment variable values. Especially in cases when you would like to easily enter a value, but in the end, you need a result in another, more complicated syntax.

The typical example of such a case is the ease of defining an environment variable with a value specified in a plain JSON object/array syntax but automatically generating its stringified version and using it in another place through another variable.

It's worth remembering that the environment variable value is represented only as a string data type. When you need to parse it as a plain JSON value in a consumer service, it has to be represented by a stringified value. Otherwise, it fails.

That's why Zerops offers the `stringify` modificator that can be used in the following ways:

```yaml
# Transforming the CONFIG environment variable into a stringified JSON value in the CONFIG_JSON variable.
envVariables:
    CONFIG: {"version": 1.0, "mode": "P", "debug": null}
    CONFIG_JSON: ${CONFIG|stringify}
# The result contained in CONFIG_JSON variable: {\"version\":1.0,\"mode\":\"P\",\"debug\":null}
```

```yaml
# Transforming the CONFIG environment variable into a stringified JSON value in the CONFIG_JSON variable.
envVariables:
    CONFIG: |
      {
        "version": 1.0,
        "mode": "P",
        "debug": null
      }
    CONFIG_JSON: ${CONFIG|stringify}
# The result contained in CONFIG_JSON variable: {\"version\":1.0,\"mode\":\"P\",\"debug\":null}
```

:::
<!-- markdownlint-enable DOCSMD004 -->

#### nginxConfig

`nginxConfig`: string (optional)

This is required only for the [PHP+Nginx](/documentation/services/runtimes/php.html#default-nginx-config) and the Nginx [Static server](/documentation/services/static-servers/nginx.html#default-nginx-config) services. The value represents the required content of the configuration `nginx.conf` file used by the Nginx server. Part of that configuration is also setting a document root value.

For example, this could be the exported value if a user accepts the default setting when creating the PHP+Nginx service in the Zerops GUI.

```yaml
services:
  - hostname: phpnginx
    type: php-nginx@8.0+1.20
    minContainers: 1
    maxContainers: 1
    nginxConfig: |
      server {
        listen 80;
        listen [::]:80;
        
        server_name _;
        # Be sure that you set up a correct document root!
        root /var/www/public;
        
        location / {
          try_files $uri /index.php$is_args$args;
        }
        
        location ~* \.php$ {
          fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
          fastcgi_split_path_info ^(.+\.php)(/.*)$;
          include fastcgi_params;
          fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
          fastcgi_param DOCUMENT_ROOT $realpath_root;
          internal;
        }
        
        access_log syslog:server=unix:/dev/log,facility=kern default_short;
        error_log syslog:server=unix:/dev/log,facility=kern;
      }
```

And this is an example for the same when creating the Nginx Static server service in the Zerops GUI.

```yaml
services:
  - hostname: nginx
    type: nginx@1.20
    minContainers: 1
    maxContainers: 1
    nginxConfig: |
      server {
          listen 80 default_server;
          listen [::]:80 default_server;
          
          server_name _;
          # Be sure that you set up a correct document root!
          root /var/www/public;
          
          location / {
              try_files $uri $uri/ =404;
          }
          
          access_log syslog:server=unix:/dev/log,facility=local1 default_short;
          error_log syslog:server=unix:/dev/log,facility=local1;
      }
```

#### verticalAutoscaling

`verticalAutoscaling`: Map[`key:value`] (optional)

A vertical autoscaling map allows the following keys: `minCpu`, `maxCpu`, `minRam`, `maxRam`, `minDisk`, `maxDisk`.

Related to all services except [Object Storage](/documentation/services/storage/s3.html).

##### minCpu

`minCpu`: integer [>= 1 and <= upper service limit] (optional)

A minimum number of virtual CPUs (vCPU) to be allocated for a given service. If the [maxCpu](#maxCpu) property is not specified, the service will be scaled to the upper service limit (which can be generally different for each service type and can be changed in the future).

##### maxCpu

`maxCpu`: integer [>= [minCpu](#minCpu) and <= upper service limit] (optional)

A maximum number of virtual CPUs (vCPU) to be allocated for a given service. If the [minCpu](#minCpu) property is not specified, the service will be scaled from the lower service limit.

Actual service vCPU lower and upper limits:

|Service vCPU   |lower limit |upper limit |
|:--------------|:-----------|:-----------|
|PostgreSQL     |1           |20          |
|MariaDB        |1           |20          |
|KeyDB          |1           |20          |
|RabbitMQ       |1           |20          |
|Shared storage |1           |20          |
|Node.js        |1           |20          |
|Golang         |1           |20          |
|PHP+Apache     |1           |20          |
|PHP+Nginx      |1           |20          |
|Static server  |1           |20          |

The lower and upper service limits of vCPU can be generally different for each service type and can be changed in the future.

##### minRam

`minRam`: float [>= lower service limit and <= upper service limit] in GB (optional)

A minimum size of RAM (in GB) to be allocated for a given service. If the [maxRam](#maxRam) property is not specified, the service will be scaled to the upper service limit (which can be generally different for each service type and can be changed in the future).

##### maxRam

`maxRam`: float [>= lower service limit and <= upper service limit] in GB (optional)

A maximum size of RAM (in GB) to be allocated for a given service. If the [minRam](#minRam) property is not specified, the service will be scaled from the lower service limit.

Actual service RAM lower and upper limits:

|Service RAM in GB |lower limit |upper limit |step |
|:-----------------|:-----------|:-----------|:----|
|PostgreSQL        |1           |32          |0.25 |
|MariaDB           |1           |32          |0.25 |
|KeyDB             |0.25        |32          |0.25 |
|RabbitMQ          |0.25        |32          |0.25 |
|Shared storage    |0.25        |32          |0.25 |
|Node.js           |0.25        |32          |0.25 |
|Golang            |0.25        |32          |0.25 |
|PHP+Apache        |0.25        |32          |0.25 |
|PHP+Nginx         |0.25        |32          |0.25 |
|Static server     |0.25        |32          |0.25 |

The lower and upper service limits of RAM can be generally different for each service type and can be changed in the future.

##### minDisk

`minDisk`: float [>= lower service limit and <= upper service limit] in GB (optional)

A minimum size of DISK (in GB) to be allocated for a given service. If the [maxDisk](#maxDisk) property is not specified, the service will be scaled to the upper service limit (which can be generally different for each service type and can be changed in the future).

##### maxDisk

`maxDisk`: float [>= lower service limit and <= upper service limit] in GB (optional)

A maximum size of DISK (in GB) to be allocated for a given service. If the [minDisk](#minDisk) property is not specified, the service will be scaled from the lower service limit.

Actual service DISK lower and upper limits:

|Service DISK in GB |lower limit |upper limit |step |
|:------------------|:-----------|:-----------|:----|
|PostgreSQL         |1           |100         |0.5  |
|MariaDB            |1           |100         |0.5  |
|KeyDB              |1           |100         |0.5  |
|RabbitMQ           |1           |100         |0.5  |
|Shared storage     |1           |100         |0.5  |
|Node.js            |1           |100         |0.5  |
|Golang             |1           |100         |0.5  |
|PHP+Apache         |1           |100         |0.5  |
|PHP+Nginx          |1           |100         |0.5  |
|Static server      |1           |100         |0.5  |

The lower and upper service limits of DISK can be generally different for each service type and can be changed in the future.

Example of the verticalAutoscaling syntax:

```yml
verticalAutoscaling:
  minVirtualCpu: 5
  maxVirtualCpu: 10
  maxRam: 19.75
  minDisk: 25.5
```

#### objectStorageSize

`objectStorageSize`: integer (optional)

This is required only for the [Object Storage](/documentation/services/storage/s3.html#required-disk-capacity) service. The value represents the required maximum amount of data in GB the Object Storage Service should be capable of holding. The preset value in the Zerops GUI is ==`2`== GB.

#### buildFromGit

`buildFromGit`: string (optional)

A public Git URL of a repository should be cloned by Zerops and used for building such a service, including the branch name. It allows for fully automatic processing of the same steps that can be taken manually by a user in the Zerops GUI (including selecting the `Build immediately after the service creation` checkbox). **There is a strict condition: The repository has to be public.**

The format of the URL should be: `https://<domain>@<branchName>`

Related only to [Node.js](/documentation/services/runtimes/nodejs.html#how-to-deploy-application-code), [Golang](/documentation/services/runtimes/golang.html#how-to-deploy-application-code), and [PHP](/documentation/services/runtimes/php.html#how-to-deploy-application-code) runtime environment services.

## Current known limitations

1. It's not possible to use private repositories in combination with the `buildFromGit` item. If you want to import a service related to a private repository, you have to connect it manually in the Zerops GUI.

2. You have to configure your [domain](/documentation/routing/using-your-domain.html), or [Zerops subdomain](/documentation/routing/zerops-subdomain.html) access directly in the Zerops GUI.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Secure access from the external Internet
Before allowing access to a service from the external Internet, it's important to be aware of potential security risks if this service **doesn't have an authentication mechanism implemented** to prevent accidental anonymous user access. Remember that it's possible to use the Zerops [zCLI VPN](/documentation/cli/vpn.html) and access a service like this securely through the URL `http://<hostname>:<port>` inside your Zerops project network without allowing external access at all.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable MD029 -->
3. When using shared storage services, neither the export nor the import doesn't support the [storage mounting points](/documentation/services/storage/shared.html#storage-mounting) in relation to [Node.js](/documentation/services/runtimes/nodejs.html#accessing-a-zerops-shared-storage), [Golang](/documentation/services/runtimes/golang.html#accessing-a-zerops-shared-storage), and [PHP](/documentation/services/runtimes/php.html#accessing-a-zerops-shared-storage) runtime environment services. It means that the export doesn't publish already mounted shared storage services, and the import doesn't allow to specify which ones should be enabled on which runtime services. Mounting them to the existing runtime services must be done later manually in the Zerops GUI.
<!-- markdownlint-enable MD029 -->

## Zerops recipes

The concept of pre-made skeletons demonstrates how to set up and use the technologies that Zerops supports. They use the import syntax to allow for the very fast creation of services utilizing such technologies.

Here are some of them:

* [recipe-adminer](https://github.com/zeropsio/recipe-adminer)

Adminer is a full-featured database management tool written in PHP. Unlike phpMyAdmin, it consist of a single file ready to deploy to the target server. Adminer is available for MySQL, MariaDB, PostgreSQL, SQLite, MS SQL, Oracle, Elasticsearch, and others via a plugin.
