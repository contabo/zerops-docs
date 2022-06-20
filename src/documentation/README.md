# Introduction

Zerops is a cloud **Platform-as-a-Service** (PaaS) made [by developers for developers](/documentation/overview/made-for-developers.html). All of Zerops' services, whether it's a [runtime](/documentation/services/runtimes.html), [static server](/documentation/services/static-servers.html), [database](/documentation/services/databases.html), [message brokers](/documentation/services/message-brokers.html) or [storage](/documentation/services/storage.html), are HA ([high availability](/documentation/ha/why-should-i-want-high-availability.html)) enabled by default and [automatically scaled](/documentation/automatic-scaling/how-automatic-scaling-works.html) and **optimized** to always run in the [most efficient and reliable way](/documentation/ha/zerops-enterprise-grade-reliability.html).

**Here are the main things Zerops offers you:**

## 1. Build, deploy, and run your applications or serve static files

A good build and deploy setup is hard, so we try to make it easy no matter your use case. You can [connect your service with a GitHub repository](/documentation/github/github-integration.html) to automatically trigger a [build and deploy](/documentation/build/how-zerops-build-works.html) pipeline using the [`zerops.yml`](/documentation/build/build-config.html) build definition file.

Alternatively, you can use [Zerops CLI](/documentation/cli/installation.html) to manually trigger [deploy](/documentation/deploy/how-deploy-works.html) of your build artefacts, or `zcli push` cli [command](/documentation/cli/available-commands.html#push) to start the build and deploy pipeline. You have the flexibility to do this locally from your terminal, from [GitHub Actions](/documentation/deploy/use-in-github-actions.html), [Gitlab CD/CI](/documentation/gitlab/gitlab-integration.html) or any other [CI service](/documentation/cli/available-commands.html#deploy).

Whether you need to serve a [JAMSTACK](https://jamstack.org/) app or run an environment, we have you covered. Serve a static Angular, Vue or React app from our [Nginx server](/documentation/services/static-servers.html), run multiple [Node.js](/documentation/services/runtimes.html#node-js), [Golang](/documentation/services/runtimes.html#golang) or [PHP](/documentation/services/runtimes.html#php) applications on Zerops [runtime services](/documentation/services/runtimes.html).

## 2. Operate fully administered databases

All the databases, message brokers, or shared storage Zerops manages are running on at least [three hardware separated nodes](/documentation/ha/why-should-i-want-high-availability.html) to ensure the highest possible data security and reliability. Each of [MariaDB (MySQL)](/documentation/services/databases/mariadb.html), [PostgreSQL](/documentation/services/databases/postgresql.html), [KeyDB (Redis)](/documentation/services/databases/keydb.html), [RabbitMQ](/documentation/services/message-brokers/rabbitmq.html), and [Shared Storage](/documentation/services/storage/shared.html) is fine-tuned to be as performant as possible at any given [automatic scaling state](/documentation/automatic-scaling/how-automatic-scaling-works.html). Everything is fully managed by Zerops with [backups](/documentation/backup-restore/snapshot-backup.html) at various geographical locations.

## 3. Manage internal and public routing

Each service is automatically set up to be able to [securely communicate](/documentation/routing/routing-between-project-services.html) with other services inside the same project using their hostnames and ports, i.e. your Golang app service (`myapp:3000`) can access a PostgreSQL database service(`mydatabase:5432`) using `postgresql://mydatabase:5432` connection string. You can manage which ports are open inside the Zerops app.

If your application needs to be publicly accessible, Zerops can easily set up access from your [domains](/documentation/routing/using-your-domain.html), as well as direct access through a [unique IPv4 or IPv6 address](/documentation/routing/unique-ipv4-ipv6-addresses.html), with the ability to setup [firewall](/documentation/routing/access-through-ip-and-firewall.html) rules. You can also enable a [Zerops subdomain](/documentation/routing/zerops-subdomain.html) if you just need to do some testing.

## 4. Automatically scale from 0 to ∞

Zerops is able to [scale your services both horizontally and vertically](/documentation/automatic-scaling/how-automatic-scaling-works.html), whether it's a runtime, database or storage. A service can easily go from **the lower initial setting** to **the upper limits** and back down in matter of seconds, all scaling operations optimized for the technology in question and its specific version.

## 5. Focus on development, we do the rest

We try not to get in your way. We know that for a good dev experience you might want a separate infrastructure for each environment (stage, prod), or even each developer, that's why Zerops places no limits on the number of projects or team members. Each service can start with minimum HW resources or be stopped entirely if you don't need them. Alternatively, it can run in HA mode and support thousands of users 24 hours a day.

We also provide a number of tools to make dev life easier, one of which is the [VPN](/documentation/cli/vpn.html) capability of our CLI, so you can [connect locally](/documentation/cli/available-commands.html#start-project-name) to any of your running services by using its Zerops hostname and a port, i.e. visiting `http://myapp:3000` or `mariadb://mydatabase:3306`.
