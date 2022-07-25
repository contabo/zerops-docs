# RabbitMQ

Zerops provides a fully managed and scaled [RabbitMQ](https://www.rabbitmq.com) service, suitable for both development and production projects using any load. It's a highly available, fault tolerant and scalable message queuing solution. It understands AMQP (Advanced Message Queuing Protocol), STOMP (Streaming Text Orientated Messaging Protocol), MQTT (Message Queuing Telemetry Transport), and other [protocols](https://www.rabbitmq.com/protocols.html). You can choose any option you wish in the knowledge that it will work.

[[toc]]

## Adding the RabbitMQ Service in Zerops

The Zerops RabbitMQ service is based on a [Linux LXD container](/documentation/overview/projects-and-services-structure.html#services-containers) with **Ubuntu** **==v18.04.06==**.

There are two possible ways of creating a new RabbitMQ service. Either manually in the [Zerops GUI](#through-the-zerops-gui-interface), or using the Zerops [import functionality](/documentation/export-import/project-service-export-import.html#how-to-export-import-a-project).

### Using the import functionality

Zerops uses a YAML definition format to describe the structure. View the complete specification of the [import/export syntax in the YAML format](/documentation/export-import/project-service-export-import.html#used-yaml-specification).

To import a RabbitMQ service, you can use something similar to the following:

```yaml
services:
  # Service will be accessible through zCLI VPN under <protocol>://mq:<port>
  - hostname: mq
    # Type and version of service used.
    type: rabbitmq@3
    # Whether the service will be run on one or multiple containers.
    # Since this is a simple example, using only one container is fine.
    mode: NON_HA
```

### Through the Zerops GUI interface

#### Which version to choose

You can currently only choose version **v3** (version 3.9.13 to be precise).

Used as the export & import types: ==`rabbitmq@3`== .

#### Hostname and ports

Choose a short and descriptive URL-friendly name, for example, **mq**. The following rules apply:

* maximum length **==25==** characters,
* only lowercase ASCII letters **==a-z==** and numbers **==0-9==**,
* **==has to be unique==** in relation to other existing project hostnames,
* the hostname **==can't be changed==** later.

The ports will automatically be set to the values of **==1883==** (the `mqtt` protocol), **==5672==** (the `amqp` protocol), **==61613==** (the `stomp` protocol), **==15672==** (web management portal),  **==15674==** (WebSocket for the `stomp` protocol), **==15675==** (WebSocket for the `mqtt` protocol) and can't be changed.

![RabbitMQ Service](./images/RabbitMQ-Internal-Ports.png "Message Broker Internal Ports")

<!-- markdownlint-disable DOCSMD004 -->
::: warning Hostname is also used as the default admin user name
The chosen **hostname** is automatically used to create an admin user account with all privileges and grant options for accessing the message broker. You can change it later if you prefer. The standard default user `guest` is removed during the installation phase of the service.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: warning Security inside a private project network
All services inside a Zerops project share a [dedicated private network](/documentation/routing/routing-between-project-services.html) and can see and reference [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) from other services. This means that, by default, nothing outside the project can access any of the services inside it.

The RabbitMQ service is configured to **allow access** for native `mqtt`, `amqp`, `stomp` protocols only from the project internal network environment. In practice, this means that this service can only be accessed either programmatically through the runtime environment services ([Node.js](/documentation/services/runtimes/nodejs.html#port), [Golang](/documentation/services/runtimes/golang.html#port), and [PHP](/documentation/services/runtimes/php.html#hostname-and-port), which should implement their authentication logic not to allow anonymous access to the RabbitMQ service) or using [zCLI](/documentation/cli/installation-authorization.html) and [vpn](/documentation/cli/vpn.html) from your local environment, nothing else.
:::
<!-- markdownlint-enable DOCSMD004 -->

##### Selective access to the internal ports

You can use the Zerops [subdomain](/documentation/routing/zerops-subdomain.html) feature to access the RabbitMQ [web management portal](https://www.rabbitmq.com/management.html) (port 15672) and subdomains with the <strong>wss</strong> protocol are intended to be used as URLs to connect to the STOMP (15674) and MQTT (15675) WebSocket servers on the RabbitMQ service. See also an explanation about [mapping to the STOMP and MQTT WebSockets server plugins](#mapping-to-stomp-and-mqtt-websockets-server-plugins) from the outside Internet.

![RabbitMQ Service](./images/RabbitMQ-Subdomain_Web_Management_Portal.png "Subdomain to Web Management Portal")

<!-- markdownlint-disable DOCSMD004 -->
::: tip Web management portal access in HA mode
If you enable the Zerops subdomain to access the portal from the Internet, you will see the URL list of the RabbitMQ cluster nodes first:

* RabbitMQ GUI Container 1 `<hostname>-<randomId>-<port>.app.zerops.io/rabbit1/#/`
* RabbitMQ GUI Container 2 `<hostname>-<randomId>-<port>.app.zerops.io/rabbit2/#/`
* RabbitMQ GUI Container 3 `<hostname>-<randomId>-<port>.app.zerops.io/rabbit3/#/`

You can click on any of these three URLs to access the web management portal on the corresponding RabbitMQ cluster node because all nodes are equivalent. The logic is precisely the same even when you access the portal through a mapped public domain.
:::
<!-- markdownlint-enable DOCSMD004 -->

The Zerops HTTP routing feature allows you also to map public domains or subdomains to the RabbitMQ [web management portal](https://www.rabbitmq.com/management.html) (port 15672), or WebSockets [STOMP](https://www.rabbitmq.com/web-stomp.html) (port 15674) and [MQTT](https://www.rabbitmq.com/web-mqtt.html) (port 15675) plugins. It enables access from public Internet through web browsers, and even transmit messages using STOMP and MQTT protocols.

![RabbitMQ Service](./images/RabbitMQ-Public_Domains_Mapping.png "Public Domains Mapping")

Zerops uses this possibilities to offer only appropriate options and SSL certificates in useful places. These ports are used to set up public Internet access as described in the following section: [From the external Internet environment](#from-the-external-internet-environment).

The following image shows the case of the `portal.ikbase.eu` public subdomain is pointed to the web management portal of the RabbitMQ service on the host `mq:15672`. The same can be done for WebSockets STOMP `stomp.ikbase.eu` -> `mq:15674` or MQTT `mqtt.ikbase.eu` -> `mq:15675` plugins.

![Public Routing](./images/RabbitMQ-Public-Access-Internal-Ports.png "Public Routing Overview")

##### Mapping to STOMP and MQTT WebSockets server plugins
<!-- markdownlint-disable DOCSMD004 -->
::: tip
Both RabbitMQ STOMP and MQTT WebSocket server plugins are located on the `/ws` path. That's why you should use the same `/ws` public path to map the public subdomains. It's also important to remember that the described Zerops mapping transforms the URL that a JavaScript client uses to `wss://stomp.ikbase.eu/ws` or `wss://mqtt.ikbase.eu/ws`. It means that the `443` port is used, not the `15674` or `15675` ports from the public Internet.

In the case of HA mode, WebSockets requests are transparently routed to the individual RabbitMQ nodes through the load balancers, which are part of the RabbitMQ cluster. Take a look at the [RabbitMQ Service in HA Mode, a deep-dive view](/documentation/overview/how-zerops-works-inside/rabbitmq-cluster-internally.html).
:::
<!-- markdownlint-enable DOCSMD004 -->

#### HA / non-HA message broker mode

When creating a new service, you can choose whether the database should be run in **HA** (High Availability) mode, using 3 containers, or **non-HA mode**, using only 1 container. ==**The chosen database mode can't be changed later.**== If you would like to learn more about the technical details and how this service is built internally, take a look at the [RabbitMQ Service in HA Mode, a deep-dive view](/documentation/overview/how-zerops-works-inside/rabbitmq-cluster-internally.html).

##### RabbitMQ in non-HA mode

* great for local development to save money,
* queues data are stored only in a single container, higher risk of loss,
* all queues data changes since the last backup are not recoverable,
* not recommended for production projects.

##### RabbitMQ in HA mode

* will run on three containers as a [cluster](https://www.rabbitmq.com/clustering.html), each on a **different physical machine**,
* therefore queues data are stored redundantly in three places, with no risk of loss,
* when one container fails, it's automatically replaced with a new one,
* come with two load balancers ([HAProxy](http://www.haproxy.org)) (no additional cost),
* recommended for production projects.

## How to access a RabbitMQ message broker

<!-- markdownlint-disable DOCSMD004 -->
::: warning Don't use additional security protocols for internal communication
The messaging service is not configured to support direct access using SSL/TLS or SSH protocols for internal communication inside a Zerops project private secured network. This is also the case for access using the Zerops [zCLI](/documentation/cli/installation.html) through a secure VPN channel.
:::
<!-- markdownlint-enable DOCSMD004 -->

### From other services inside the project

Other services can access the broker using its **hostname** and **port** environment variables (the primary native `amqp` communication protocol is applied), as they are part of the same private project network (for example, `amqp://mq:5672`). It’s highly recommended that you utilize the **==connectionString==** environment variable that Zerops creates automatically for the message broker in non-HA mode. See also the explanation of environment variables specifics for HA mode below. More information related to **connectionString** can be found in the dedicated [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) section. See also a list of all automatically generated [environment variables](/documentation/environment-variables/helper-variables.html#rabbitmq) for the RabbitMQ service.

If you need to use the `mqtt` or `stomp` protocols, use the **==connectionStringMqtt==** or **==connectionStringStomp==** environment variables (**portMqtt** and **portStomp** ones also exist).

![RabbitMQ Service](./images/RabbitMQ-Message-Broker-Environment-Variables.png "Message Broker Environment Variables")

For more flexibility with future potential hostname changes, it's always recommended to use them indirectly via [custom environment variables](/knowledge-base/best-practices/how-to-use-environment-variables-efficiently.html) (referencing implicit Zerops environment [variables](/documentation/environment-variables/helper-variables.html#rabbitmq)) in each project service separately. This allows you to eliminate all direct dependencies in the application code, which in turn provides simplification and increased flexibility. Another reason not to hard-code the values inside your applications is that it can be dangerous because it is easy to commit them (like your credentials) into a repository, potentially exposing them to more people than intended.

### From other Zerops projects

Zerops always sets up a [private dedicated network](/documentation/overview/projects-and-services-structure.html#project) for each project. From this point of view, cross project communication can be done precisely in the same ways described in the section [From your public domains (common Internet environment)](#from-your-public-domains-common-internet-environment). There isn't any other specific way. These projects are not directly interconnected.

### From your local environment

The local environment offers ==**not only options for local development**== but also a general ability to ==**manage all Zerops development or production services**== , using zCLI VPN.

To connect to the message broker from your local workspace, you can utilize the [VPN](/documentation/cli/vpn.html) functionality of our [Zerops zCLI](/documentation/cli/installation.html), as previously mentioned. This allows you to access the message broker the same way other services inside the project can, but unlike those services, you cannot use references to the environment variables. Therefore, if you need some of them you should copy the values manually through the **How To Access** / **Access details** section of the service detail in your application and use them in your private local configuration strategy.

![RabbitMQ Service](./images/RabbitMQ-Message-Broker-Access-Details.png "Message Broker Access Details")

After the zCLI [login](/documentation/cli/available-commands.html#login) and establishing a [VPN channel](/documentation/cli/available-commands.html#vpn-start-project-name), you can use the URL `http://mq:15672` to access the RabbitMQ web management portal.

![RabbitMQ Service](./images/RabbitMQ-Web-Management-Portal.png "Message Broker Web Management Portal")

An alternative method is to download [rabbitmqadmin](https://www.rabbitmq.com/management-cli.html) from `http://mq:15672/rabbit1/cli/rabbitmqadmin` or directly from [GitHub](https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/v3.9.13/deps/rabbitmq_management/bin/rabbitmqadmin). This is a CLI version that can perform the same actions as the web-based management portal. The tool requires that a supported version of Python 3 is installed.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a custom path prefix for all HTTP requests
Zerops uses the custom path prefix configuration `management.path_prefix = /rabbit1` for a standalone node in non HA mode. The configuration cluster's node values in HA mode are `/rabbit1`, `/rabbit2`, and `/rabbit3`.
:::
<!-- markdownlint-enable DOCSMD004 -->

### From your public domains (common Internet environment)

You can't access the RabbitMQ service native protocols (`amqp`, `mqtt`, `stomp`) directly in any way. You have to use one of the runtime environment services ([Node.js](/documentation/services/runtimes/nodejs.html), [Golang](/documentation/services/runtimes/golang.html), or [PHP](/documentation/services/runtimes/php.html)) and go indirectly through them in a programmatic way. They should implement either their custom authentication logic and access the RabbitMQ service under a system shared account or use personalized authentication mechanisms that RabbitMQ supports internally.

To understand this better, take a look at the following section: [With external access](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html#with-external-access) of **Typical schemas of Zerops Projects**.

To take advantage of the direct access to the RabbitMQ [web management portal](https://www.rabbitmq.com/management.html) (port 15672) or transmit messages using WebSockets and the RabbitMQ [STOMP](https://www.rabbitmq.com/web-stomp.html) (port 15674) and [MQTT](https://www.rabbitmq.com/web-mqtt.html) (port 15675) plugins, see the [Selective access to the internal ports](#selective-access-to-the-internal-ports) section.

## Default RabbitMQ user and password

Zerops automatically creates a user with all privileges and grant options when creating the service, where the name of **==user==** is based on the selected **hostname**, and the **==password==** is randomly generated. These are saved to the environment variables **user** and **password** and can be referenced from other services the same way as with a **connectionString**, for example.

<!-- markdownlint-disable DOCSMD004 -->
::: warning Zerops doesn’t keep both places in sync
If you change your password inside the RabbitMQ broker directly, the change is not reflected in the environment variable and vice versa. It’s up to you to keep these up to date through the **Service env. Variables** section of the service detail in your application.

![RabbitMQ Service](./images/RabbitMQ-Message-Broker-Access-Change-Password.png "Message Broker Access Change Password")
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: tip Default Zerops maintenance user
For system maintenance reasons, the `zps` user is also automatically created with all privileges. It's important not to change this in any way. Otherwise, there is a risk of disrupting the correct functionality, especially in HA mode. The password for this user can be taken via the `zeropsPassword` environment variable (not presented in the Zerops GUI).

If you log in as the `zps` user and make inappropriate changes to the system configuration (especially in the HA cluster setting), then you are fully responsible for any system or application failures that may lead to complete data loss.
:::
<!-- markdownlint-enable DOCSMD004 -->

## Default hardware configuration and autoscaling

* Each RabbitMQ container (1 in non-HA, 3 in HA) starts with the technological minimum of vCPUs (allowing a meaningful initial start and response time), 0.25 GB RAM, and 1 GB of disk space.
* Zerops will only automatically scale the resources [vertically](/documentation/automatic-scaling/how-automatic-scaling-works.html#vertical-scaling) (both in non-HA and HA mode).
* The [horizontal autoscaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) in HA mode is not applied because of optimal performance.

## Backup and Restore

Every RabbitMQ node has a data directory that stores all the information that resides on that node. A data directory contains two types of data: **definitions** (metadata, schema/topology) and **messages data**. Definitions (mostly static) can be [exported](https://www.rabbitmq.com/definitions.html#export) and [imported](https://www.rabbitmq.com/definitions.html#import) as JSON files from/to any cluster node with the same result. Messages data can be backed up/restored only on a stopped node. In the case of a cluster, the entire cluster has to be stopped to take a backup/restore.

### How to backup / restore definitions

Definitions are stored in an internal database and replicated across all cluster nodes. Every node in a cluster has its own replica of all definitions. When part of definition changes, the update is performed on all nodes in a single transaction.

#### Using rabbitmqadmin CLI

Connect to your Zerops project using [zCLI](/documentation/cli/installation.html) & [VPN](/documentation/cli/vpn.html), and then you can use the URL ==`http://mq:15672/rabbit1/cli/rabbitmqadmin`== to download **rabbitmqadmin** CLI. Alternatively, you can do the same from [GitHub](https://raw.githubusercontent.com/rabbitmq/rabbitmq-server/v3.9.13/deps/rabbitmq_management/bin/rabbitmqadmin). The tool requires a supported version of Python 3 to be installed.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Setting permissions for rabbitmqadmin
Remember to set the executable rights on the Linux or Mac platform using `chmod 755 rabbitmqadmin`.
:::
<!-- markdownlint-enable DOCSMD004 -->

##### Logical definitions backup

```bash
rabbitmqadmin -H [hostname] -P [port] -u [user] -p [password] --path-prefix=[prefix] export [filename].json
```

And when used values:

* hostname = **==mq==** (specified when RabbitMQ Service was created)
* port = **==15672==** (automatically set as a default value)
* user = **==mq==** (automatically created with the same name as the hostname)
* password = **==xxxxxxxxxxxxxxxx==** (automatically created as a random value)
* prefix = ==**/rabbit1**== (also ==**/rabbit2**== , or ==**/rabbit3**== in HA mode)
* filename = **==definitions==** (specified filename to store the definitions backup)

```bash
rabbitmqadmin -H mq -P 15672 -u mq -p xxxxxxxxxxxxxxxx --path-prefix=/rabbit1 export definitions.json
```

##### Logical definitions restore

```bash
rabbitmqadmin -H [hostname] -P [port] -u [user] -p [password] --path-prefix=[prefix] import [filename].json
```

And when used values:

* hostname = **==mq==** (specified when RabbitMQ Service was created)
* port = **==15672==** (automatically set as a default value)
* user = **==mq==** (automatically created with the same name as the hostname)
* password = **==xxxxxxxxxxxxxxxx==** (automatically created as a random value)
* prefix = ==**/rabbit1**== (also ==**/rabbit2**== , or ==**/rabbit3**== in HA mode)
* filename = **==definitions==** (specified filename to store the definitions backup)

```bash
rabbitmqadmin -H mq -P 15672 -u mq -p xxxxxxxxxxxxxxxx --path-prefix=/rabbit1 import definitions.json
```

#### Using the RabbitMQ web management portal

![RabbitMQ Service](./images/RabbitMQ-Web-Management-Portal-Backup-Restore.png "Message Broker Web Management Portal Backup/Restore")

### How to backup / restore messages data

Messages are often short-lived and possibly transient, backing them up from under a running node is highly discouraged and can lead to an inconsistent snapshot of the data. For these reasons, it's not recommended to backup/restore any messages from the user side. Zerops takes [daily snapshots](/documentation/backup-restore/snapshot-backup.html) instead.
