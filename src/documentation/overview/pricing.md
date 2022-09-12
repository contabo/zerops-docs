# Pricing

![Pricing](/pricing.png "Pricing")

Zerops has three high-level pricing concepts. Credit that you can top up manually or automatically, deducted project and service costs, and paid add-ons which work as a subscription for 30 days following their activation. Nothing else, no hidden or artificial costs.

## Team members

Everything starts with an idea. To make it a reality, you need a team of co-developers. We [encourage](/documentation/overview/made-for-developers.html#each-developer-should-have-his-own-account-no-artificial-pricing-boosting) each developer to get their own Zerops account in order to make full use of our [local development tools](/documentation/cli/vpn.html). In Zerops, creating a [team](/documentation/overview/users.html#your-account) big enough to match your needs is **unlimited and free**.

## Credit and free Zerops credit

To begin with, you'll get \$20 worth of Zerops credit. This is a stress-free way to start off and gives you the freedom to explore all the features which could provide added value for you and your team. This credit can be used for everything including specific add-ons, such as the dedicated IPv4 address. After using up the \$20, you can manually top up your credit using a credit card or set up weekly/monthly automatic payments. At the same time, you can also set up a daily project cost limit to ensure that you don't run over your budget. The VAT rate is 21% for EU countries. We will also notify you by e-mail when your credit is too low (i.e., less than calculated 10%).

## Projects

Each [project](/documentation/overview/projects-and-services-structure.html#project) you add activates the **Project Basic Package for \$5 per 30 days**. This includes a unique IPv6 address, core services with built-in logging, an L7 balancer that takes care of SSL certification, and daily backups kept two days back. Further add-ons can be activated for each project, depending on your use case, whether it's a production project or any other environment.

Any team member can create as many projects as necessary to cover each environment, even for a single hour. Your credit is charged for a project basic package once the first project's service is added, but the unused portion of the amount is refundable if the project is deleted. The calculation is based on a daily logic, and it also respects the time of activation. That's why a project created on Monday at 5 p.m. and deleted on Tuesday at 11 a.m., costs only \$5 / 30 \* 1, and the rest is refunded back to your credit balance. Also, creating a project and deleting it after one hour of activity, and repeating this pattern X-times during the same day, leads to the cost of X \* \$5 / 30 * 1.

## Services

Each [service](/documentation/overview/projects-and-services-structure.html#service) (except S3 object storage) that you include into the [project](/documentation/overview/projects-and-services-structure.html#project) can be added either as non-HA [High Availability] (useful for development/test projects), it will run on a single container, or HA (a must-have for production projects), it will run on at least [three containers](/documentation/ha/why-should-i-want-high-availability.html). The hourly cost will change according to your choice. Each container starts with 1 vCPU, 250 MB RAM (scaling up by 250 MB increments), and 5 GB of SSD disk space (scaling up by 0.5 GB increments). All hardware resources can be automatically scaled both horizontally (increasing the number of containers) and vertically (adding resources).

**HA mode of database** ([MariaDB (MySQL)](/documentation/services/databases/mariadb.html#ha-non-ha-database-mode), [PostgreSQL](/documentation/services/databases/postgresql.html#ha-non-ha-database-mode), [KeyDB (Redis)](/documentation/services/databases/keydb.html#ha-non-ha-database-mode)), **message broker** ([RabbitMQ](/documentation/services/message-brokers/rabbitmq.html#ha-non-ha-message-broker-mode)), and [Shared Storage](/documentation/services/storage/shared.html#ha-non-ha-shared-storage-mode) **services can't be changed later to non-HA mode or vice versa.** However, we plan to implement service cloning, allowing you safe migration.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Cost of stopped services
Remember that disk space cost is still being calculated and charged if you stop any service because it occupies the reserved resources.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable MD001 -->
#### S3 Object storage service
<!-- markdownlint-enable MD001 -->

Unlike other services, S3 object storage is rather specific. It is provided by a permanently running HA service based on a cluster of containers among which data is replicated and is guaranteed its synchronicity. Therefore, adding this service to the project does not create any new containers but creates a new user account providing access specifically to the storage. The required storage size then appears as a bucket. Deleting this service removes that account as well as all of its data.

#### Hardware resources cost and autoscaling

All services (except S3 object storage) are [automatically scaled](/documentation/automatic-scaling/how-automatic-scaling-works.html) both [horizontally](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) (by increasing or decreasing the number of containers) and [vertically](/documentation/automatic-scaling/how-automatic-scaling-works.html#vertical-scaling) (by adding or removing vCPUs, RAM size or disk space). The S3 object storage service can only be scaled manually in the GUI and solely in the sense of its object store space (with scaling up/down by 1 GB increments), respecting the fact that vCPUs and RAM size are irrelevant here from the project point of view. Scaling happens in a matter of seconds and is continually optimized to run using the [best possible settings](/documentation/automatic-scaling/how-automatic-scaling-works.html#performance-tunning). Your credit is charged hourly for each resource on each container, and the cost calculation considers the ratio of changes during that time.

|Hardware       |Scaling| [30 days]|   [day]|  [hour]|          Multiplier|
|:--------------|------:|---------:|-------:|-------:|-------------------:|
|vCPU           | 1 unit|  **$1.0**|$0.03333|$0.00139|number of containers|
|RAM            | 250 MB|**$1.125**|$0.03750|$0.00156|number of containers|
|Disk space     | 0.5 GB|  **$0.1**|$0.00333|$0.00014|number of containers|
|S3 Object store| 1.0 GB| **$0.02**|$0.00067|$0.00003|                   1|

## Project add-ons

You can activate the following paid add-ons:

- [a unique IPv4 address](/documentation/routing/unique-ipv4-ipv6-addresses.html): **\$4 per 30 days**

## Examples (on a regular basis)

<!-- markdownlint-disable MD001 -->
#### Production application
<!-- markdownlint-enable MD001 -->

|Package / Service|Hardware resources              |     [hour]|      [day]|  [30 days]|
|:----------------|:-------------------------------|----------:|----------:|----------:|
|basic package    |                                |    $0.0069|    $0.1667|      $5.00|
|add-on IPv4      |                                |    $0.0056|    $0.1333|      $4.00|
|HA Nginx         |3x vCPU + 3x250 MB + 3x5 GB Disk|    $0.0130|    $0.3125|      $9.38|
|HA NodeJS        |3x vCPU + 3x500 MB + 3x5 GB Disk|    $0.0177|    $0.4250|     $12.75|
|HA PostgreSQL    |3x vCPU + 3x500 MB + 3x5 GB Disk|    $0.0177|    $0.4250|     $12.75|
|                 |                      **Total**:|**$0.0609**|**$1.4625**| **$43.88**|

#### Development / Stage application

|Package / Service|Hardware resources              |     [hour]|      [day]|  [30 days]|
|:----------------|:-------------------------------|----------:|----------:|----------:|
|basic package    |                                |    $0.0069|    $0.1667|      $5.00|
|non-HA Nginx     |1x vCPU + 1x250 MB + 1x5 GB Disk|    $0.0043|    $0.1042|      $3.13|
|non-HA NodeJS    |1x vCPU + 1x250 MB + 1x5 GB Disk|    $0.0043|    $0.1042|      $3.13|
|HA PostgreSQL    |3x vCPU + 3x250 MB + 3x5 GB Disk|    $0.0130|    $0.3125|      $9.38|
|                 |                      **Total**:|**$0.0286**|**$0.6875**| **$20.63**|

___
> *The list of add-ons will grow in time. Next up: longer-stored and more frequent backups.*
