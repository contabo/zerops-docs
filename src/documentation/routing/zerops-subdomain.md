# Zerops Subdomain for Previews

Each [service](/documentation/overview/projects-and-services-structure.html#service) running on a [HTTP protocol](/documentation/routing/routing-between-project-services.html) (mostly [runtimes](/documentation/services/runtimes.html)) is assigned a unique subdomain. You can enable this subdomain within the **Internal Ports & Public Routing** tab in the service detail.

![zerops subdomain](/zerops-subdomain.png "zerops subdomain")

The Zerops subdomain can't be modified at the moment.

The Zerops subdomain is also stored under a `zeropsSubdomain` key in [environment variables](/documentation/environment-variables/overview.html#referencing-environment-variables) of the given service.

<!-- markdownlint-disable DOCSMD004 -->
::: warning NOT FOR PRODUCTION
You **shouldn't** use Zerops subdomain as CNAME for production as it will prevent autoscaling, since the traffic goes through a global Zerops L7 HTTP Balancer and not directly to your [project](/documentation/overview/projects-and-services-structure.html#project) infrastructure. Use an [IPv6 address](/documentation/routing/unique-ipv4-ipv6-addresses.html) instead or request an [IPv4 address](/documentation/routing/unique-ipv4-ipv6-addresses.html).

You should also be aware of the file **HTTP UPLOAD** transfer limit, set to **20 MB** when accessing Zerops subdomains. When this limit is reached, the operation is terminated.
:::
<!-- markdownlint-enable DOCSMD004 -->
