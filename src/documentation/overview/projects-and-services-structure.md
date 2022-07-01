# Project & Services Structure

## Project

Zerops structure has three levels. At the top is a project, which can have services made of containers. It can, for example, consist of a NodeJS [runtime environment](/documentation/services/runtimes.html), a PostgreSQL [database](/documentation/services/databases/postgresql.html), and an S3 [object storage](/documentation/services/storage.html) service.

You can create a separate project for any environment (development, stage, production) or even for each developer if you so wish. Alternatively, you can benefit from a single project shared among all developers reducing the overall cost. Either way, developers can still utilize our powerful [dev tools](/documentation/cli/vpn.html).

All services within a project share a [dedicated private network](/documentation/routing/routing-between-project-services.html) and can see and reference [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) from other services.

**By default, nothing outside the project can access any of the services inside.** Each [runtime environment](/documentation/services/runtimes.html) or [static web server](/documentation/services/static-servers.html) services can be made publicly accessible through Zerops [subdomains](/documentation/routing/zerops-subdomain.html) or [domains](/documentation/routing/using-your-domain.html), pointing your DNS records to the [IPv6](/documentation/routing/unique-ipv4-ipv6-addresses.html) or [IPv4](/documentation/overview/pricing.html#project-add-ons) addresses assigned to the project, or by setting up direct access by [opening public ports](/documentation/routing/access-through-ip-and-firewall.html) on the assigned IP addresses. A built-in optional [firewall](/documentation/routing/access-through-ip-and-firewall.html) can manage the direct access by defining a list of allowed and denied IP addresses for each open public port. For the remaining services ([databases](/documentation/services/databases.html), [message-brokers](/documentation/services/message-brokers.html)), you have to use our [dev tools](/documentation/cli/vpn.html) (VPN access and Zerops CLI). [Storage](/documentation/services/storage.html) services have a specific status and need to be approached with knowledge of their properties.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Communication between different Zerops projects
Any communication between the various Zerops projects at service level is in principle identical to accessing directly from the Internet (the same features and rules), and there are no exceptions or specifics. More about how the Zerops project works with [external access](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html#with-external-access).
:::
<!-- markdownlint-enable DOCSMD004 -->

The screenshot below shows an example of a project. The project is called **zerops-web-prod**, and it consists of 2 runtimes (Node.js and Golang), a static web server (Nginx), and a database (PostgreSQL), and object storage (S3 compatible) services. It is made accessible to the public through a domain.

<!-- markdownlint-disable DOCSMD004 -->
:::: tabs
::: tab Zerops Web Project in GUI
![Zerops Web Project](./images/Zerops-Web-Production.png "Zerops Web Project in GUI")
:::
::: tab Schema of Zerops Web Project
![Zerops Web Project](./images/Zerops-Web-Production-Schema.png "Schema of Zerops Web Project")
:::
::::
<!-- markdownlint-enable DOCSMD004 -->

Each project has the following items included in its [Basic Package](/documentation/overview/pricing.html#projects):

* 1 unique IPv6 address
* project balancers & core controllers
* firewall
* 100 build minutes
* daily backup
* 10 GB of outbound traffic

More details can be found in the [pricing](/documentation/overview/pricing.html) document.

## Services & Containers

Services are the most important part of Zerops. Each service consists of one, or multiple [Linux Containers](https://linuxcontainers.org/#LXD) (depending on your HA preference), each container running a Zerops managed image of technology, whether it’s a [runtime environment](/documentation/services/runtimes.html), a [static web server](/documentation/services/static-servers.html), a [database](/documentation/services/databases.html), a [storage](/documentation/services/storage.html), or a [message broker](/documentation/services/storage.html). Each service has a hostname and an N number of ports. It can be made public via [Zerops subdomains](/documentation/routing/zerops-subdomain.html) or [public domains](/documentation/routing/using-your-domain.html) (HTTP/S communication) or direct [IPv6/IPv4 addresses and ports](/documentation/routing/access-through-ip-and-firewall.html) mapping.

![Services](./images/Zerops-Services-Catalogue.png "Available Services")

A project can contain an [unlimited number of services](/documentation/overview/made-for-developers.html#each-developer-should-have-his-own-account-no-artificial-pricing-boosting). Depending on the type of service, they are either managed fully (scaling, repairing, routing), or partially by Zerops while giving you intuitive management functionality through the Zerops app.

### Runtime environments

[Node.js](/documentation/services/runtimes/nodejs.html), [Golang](/documentation/services/runtimes/golang.html), [PHP](/documentation/services/runtimes/php.html)

### Static server services

[Nginx](/documentation/services/static-servers/nginx.html)

### Database services

[MariaDB (MySQL)](/documentation/services/databases/mariadb.html), [PostgreSQL](/documentation/services/databases/postgresql.html), [KeyDB (Redis)](/documentation/services/databases/keydb.html)

### Message broker services

[RabbitMQ](/documentation/services/message-brokers.html#rabbitmq)

### Storage services

[Shared storage](/documentation/services/storage.html#shared-storage), [S3 compatible Object Storage](/documentation/services/storage/s3.html)
