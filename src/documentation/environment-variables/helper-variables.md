# Automatically generated helper variables

Zerops automatically adds a number of utility variables, for both internal and your own
use. They cannot be deleted, their keys cannot be changed and those marked with 🔒 cannot have their value edited.

**See variable by service type:**

[[toc]]

## Node.js, Golang

* `hostname`

🔒 Service hostname.

* `zeropsSubdomain`

🔒 Unique subdomain of Zerops, i.e. `my-app-30001.app.zerops.io`, this variable is always available, but subdomain access has to be enabled inside the service routing settings.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

## PHP/Nginx

* `hostname`

🔒 Service hostname.

* `zeropsSubdomain`

🔒 Unique subdomain of Zerops, i.e. `my-app-30001.app.zerops.io`, this variable is always available, but subdomain access has to be enabled inside the service routing settings.

* `nginxConfig`

🔒 Content of the server configuration located in most cases at `/etc/nginx/sites-available` used to define how the request should be handled by the php runtime.

* `documentRoot`

Folder from which php scripts should be executed. Deployed files are unpacked at `/var/www`.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

## PHP/Apache

* `hostname`

🔒 Service hostname.

* `zeropsSubdomain`

🔒 Unique subdomain of Zerops, ie `my-app-30001.app.zerops.io`, this variable is always available, but subdomain access has to be enabled inside the service routing settings.

* `documentRoot`

Folder from which php scripts should be executed. Deployed files are unpacked at `/var/www`.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

## MariaDB (MySQL)

* `port`

🔒 Port on which the database server runs.

* `hostname`

🔒 Service hostname.

* `connectionString`

🔒 Connection string.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

* `user`

Database admin username (changing it won't affect the value in the database settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

* `password`

Database admin password (changing it won't affect the value in the database settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

## PostgreSQL

* `port`

🔒 Port on which the database server runs in non-HA mode and also the port on which the current database primary server runs in HA cluster mode, and which has to be used to create a connection through which all data modification requests (SQL statements INSERT, UPDATE, DELETE) must be sent.

* `portReplicas`

🔒 It can be used to create a connection through which only data reading requests (SQL statement SELECT) should be sent in HA cluster mode (any cluster member can process such requests).

* `hostname`

🔒 Service hostname.

* `connectionString`

🔒 Connection string in non-HA mode and also the connection string in HA cluster mode to create a connection through which all data modification requests (SQL statements INSERT, UPDATE, DELETE) must be sent (only the current primary member processes such requests).

* `connectionStringReplicas`

🔒 Connection string in HA cluster mode to create a connection through which only data reading requests (SQL statement SELECT) should be sent (any cluster member can process such requests).

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

* `user`

Database admin username (changing it won't affect the value in database settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

* `password`

Database admin password (changing it won't affect the value in database settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

<!--
## Elasticsearch

* `port`

🔒 Port on which the database server runs.

* `hostname`

🔒 Service hostname.

* `connectionString`

🔒 Connection string.
-->

## Static Nginx Server

* `port`

🔒 Port on which the static server runs.

* `hostname`

🔒 Service hostname.

* `nginxConfig`

🔒 Content of the server configuration located in most cases at `/etc/nginx/sites-available` used to define how the request should be handled by the php runtime.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

## KeyDB (Redis)

* `port`

🔒 Port on which the database server runs.

* `hostname`

🔒 Service hostname.

* `connectionString`

🔒 Connection string.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

## RabbitMQ

* `port`

🔒 Port on which the message broker server is accessible on the `amqp` protocol.

* `portMqtt`

🔒 Port on which the message broker server is accessible on the `mqtt` protocol.

* `portWsMqtt`

🔒 Port on which the message broker server is accessible on the web socket `mqtt` protocol.

* `portStomp`

🔒 Port on which the message broker server is accessible on the `stomp` protocol.

* `portWsStomp`

🔒 Port on which the message broker server is accessible on the web socket `stomp` protocol.

* `managementPort`

🔒 Port to the web management portal service.

* `hostname`

🔒 Service hostname.

* `connectionString`

🔒 Connection string for the `amqp` protocol.

* `connectionStringMqtt`

🔒 Connection string for the `mqtt` protocol.

* `connectionStringStomp`

🔒 Connection string for the `stomp` protocol.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.

* `user`

Message broker admin username (changing it won't affect the value in engine settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

* `password`

Message broker admin password (changing it won't affect the value in engine settings, that needs to be updated manually, see [VPN](/documentation/cli/vpn.html)).

## Object Storage

* `apiUrl`

🔒 URL used to access the S3 compliant storage object API.

* `accessKeyId`

🔒 Access key to grant permission to use the object storage.

* `secretAccessKey`

🔒 Secret key to grant permission to use the object storage.

* `quotaGBytes`

🔒 Upper bound for the maximum storage consumption.

* `projectId`

🔒 Project's id to which the service belongs and which uniquely identifies it.

* `serviceId`

🔒 Service's id that uniquely identifies it.
