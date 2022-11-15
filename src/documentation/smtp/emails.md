# SMTP configuration and email sending

The default configuration of any Zerops project [private network](/documentation/routing/routing-between-project-services.html) blocks the SMTP 25 port from sending emails. The reason is primarily security and preventive protection against potential misuse of the offered [$20 credit](/documentation/overview/pricing.html#credit-and-free-zerops-credit) for the initial trial of Zerops for free for anyone.

In practice, all **outgoing TCP ports 1 - 1024 except 22, 80, and 443**, and also all **outgoing UDP ports 1 - 65635** are blocked by the project firewall. This setting, therefore, eliminates using of the SMTP port 25 for outgoing email messages.

## Zerops project's FW setting

```yaml
tcp dport 22 accept
tcp dport 80 accept
tcp dport 443 accept

tcp dport 1-1024 drop
udp dport 1-65535 drop
```

You will be able to use the SMTP port 25 without restrictions when your Zerops account is paid, but not when using a free credit. Actually, it would be best if you asked for this unblocking the Zerops team through our [Discord forum](https://discord.com/invite/WDvCZ54) or by sending an email to [team@zerops.io](mailto:team@zerops.io).

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using an IPv4 address
You'll be able to send emails over an IPv4 address even if you don't have anyone assigned to your project. In such a case, our NAT server will be used.
:::
<!-- markdownlint-enable DOCSMD004 -->

<!-- markdownlint-disable DOCSMD004 -->
::: tip Using a shadow SMTP server
To be able to simulate SMTP traffic inside your project's network when developing or testing your applications, it's possible to install a shadow SMTP server as a service directly in your Zerops project. There are several possibilities in the OSS world on how to do it, and one of them is the [mailhog](https://github.com/mailhog/MailHog) project. You can read more about it in the article [Build a Ghost site in Zerops in a matter of minutes](https://medium.com/@mlapis/build-a-ghost-site-in-zerops-in-a-matter-of-minutes-e387a6da4b62).
:::
<!-- markdownlint-enable DOCSMD004 -->
