# SMTP configuration and email sending

The default configuration of any Zerops project [private network](/documentation/routing/routing-between-project-services.html) blocks the SMTP 25 (legacy unsecured) and 587 (modern secure) ports from sending emails for security reasons. Contact us at [team@zerops.io](mailto:team@zerops.io) to manually enable the ports.

In practice, all **outgoing TCP ports 1 - 1024 except 22, 80, and 443**, and also all **outgoing UDP ports 1 - 65635** are blocked by the project firewall. This setting, therefore, eliminates using of the SMTP port 25 or 587 for outgoing email messages.

## Zerops project's FW setting

```yaml
tcp dport 22 accept
tcp dport 80 accept
tcp dport 443 accept

tcp dport 1-1024 drop
udp dport 1-65535 drop
```

If the SMTP ports 25 and 587 are enabled, you have two principal options for configuring the applications from a technical point of view so that they can send emails:

* Directly connect to the destination SMTP server according to the DNS resolution of the recipient's email address.
* Configure a connection to any public SMTP server with valid authentication credentials and send emails using it.

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using an IPv4 address
You'll be able to send emails over an IPv4 address even if you don't have anyone assigned to your project. In such a case, our NAT server will be used.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a shadow SMTP server
To simulate SMTP traffic inside your project's network when developing or testing your applications, it's possible to install a shadow SMTP server as a service directly in your Zerops project. There are several possibilities in the OSS world on how to do it, and one of them is the [MailHog](https://github.com/mailhog/MailHog) project. You can use the Zerops [import script](/documentation/export-import/project-service-export-import.html) below to create a service in a minute. The entire configuration is done through environment variables. As you can see, the service can bind to any container IP address under the hostname **mailhog**, and the defined SMTP port is **1025** instead of the blocked one **25**. From outside the Internet, you can access it from a browser using port **8025** through the enabled Zerops [subdomain](/documentation/routing/zerops-subdomain.html).
:::
<!-- markdownlint-enable DOCSMD004 -->

## Zerops import script to create a mailhog service

```yaml
services:
  - hostname: mailhog
    type: go@1
    envVariables:
      MH_API_BIND_ADDR: 0.0.0.0:8025
      MH_CORS_ORIGIN: "*"
      MH_HOSTNAME: mailhog
      MH_SMTP_BIND_ADDR: 0.0.0.0:1025
      MH_UI_BIND_ADDR: 0.0.0.0:8025
    ports:
      - port: 8025
        httpSupport: true
      - port: 1025
    minContainers: 1
    maxContainers: 1
    buildFromGit: https://github.com/zeropsio/recipe-mailhog@main
    enableSubdomainAccess: true
```
