# VPN to securely connect to your services

The [zcli](/documentation/cli/installation-authorization.html) command-line client has a WireGuard based VPN built in. This means you can connect inside the projects network and access your services locally under their [hostname and ports](/documentation/routing/routing-between-project-services.html).

[**WireGuard**](https://www.wireguard.com) needs to be installed for **zcli vpn** to work. The installation guide for each system can be found on the [WireGuard website](https://www.wireguard.com/install/). The VPN channel allows you also to connect to your Zerops project's runtime and static server containers through [SSH tunnels](/documentation/cli/ssh.html).

<!-- markdownlint-disable DOCSMD004 -->
::: warning Other WireGuard running processes
The command [zcli vpn start](/documentation/cli/available-commands.html#vpn-start-project-name-or-project-id) should only be run in an environment where no other WireGuard processes are running, otherwise we do not guarantee proper functionality.
:::
<!-- markdownlint-enable DOCSMD004 -->

## Use cases

- Create a clone of your project as your personal development environment. Use **vpn** to access your databases locally without having to have them run locally, which is especially hard if you are developing [high availability](/documentation/ha/why-should-i-want-high-availability.html) enabled applications.

- Use to [restore](/documentation/backup-restore/backup-restore-databases.html) data to your databases.

- Use to locally consume API from applications [deployed](/documentation/build/how-zerops-build-works.html) to Zerops (CMS, REST APIs).
