# RabbitMQ Service in HA Mode, a deep-dive view

The following picture shows a simplified schema of a standalone [Zerops RabbitMQ Service](/documentation/services/message-brokers/rabbitmq.html) in HA mode without external access. This means no access from outside of the Zerops project infrastructure, such as the Internet. Native access is only possible through a private network using a [default ports](/documentation/services/message-brokers/rabbitmq.html#hostname-and-ports) **==1883==**, **==5672==**, **==61613==**, **==15672==**, **==15674==**, or **==15675==** from the [Zerops Project Core Service](/documentation/overview/how-zerops-works-inside/typical-schemas-of-zerops-projects.html) itself, which is at the heart of each user-defined Zerops project.

## Simplified schema (no external access)

![Zerops RabbitMQ Service](./images/Zerops-RabbitMQ-Service-Base.png "Zerops RabbitMQ Service")

[Zerops RabbitMQ Service](/documentation/services/message-brokers/rabbitmq.html) shares the [same pricing structure](/documentation/overview/pricing.html#services) as other Zerops services. Two instances of a **[load balancer (HAProxy)](http://www.haproxy.org)** (both in the active state) play a crucial role and control the routing of user requests in tandem. They always direct requests to the least busy instance of the RabbitMQ broker service. All nodes in a [RabbitMQ cluster](https://www.rabbitmq.com/clustering.html#peer-equality) are equal peers: there are no special nodes in RabbitMQ core. An independent **scaling controller** monitors and controls [vertical scaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#vertical-scaling) (vCPU, RAM, Disk) for all load balancers, RabbitMQ message broker containers, and [horizontal scaling](/documentation/automatic-scaling/how-automatic-scaling-works.html#horizontal-scaling) (number of containers). An independent **repair controller** is then responsible for removing any containers that exhibit abnormal behavior and subsequently replacing them with new ones.

Every queue in RabbitMQ has a primary replica. That replica is called a **queue leader**. All queue operations go through the leader replica first and are then synchronously replicated to followers. This is necessary to guarantee FIFO ordering of messages.

Both load balancers or RabbitMQ message broker containers are located on **different physical computers** to prevent service outages in the event of a fatal failure on a single physical computer. Below you can see the same schema as before, this time including all relationships between the internal parts.

## Extended schema (with external access)

![Zerops RabbitMQ Service](./images/Zerops-RabbitMQ-Service-Detail.png "Zerops RabbitMQ Service")

<!-- markdownlint-disable DOCSMD004 -->
::: tip Default choice for a replicated queue type
[Quorum queues](https://www.rabbitmq.com/quorum-queues.html) should be the default choice for a replicated queue type. [Classic queue mirroring](https://www.rabbitmq.com/ha.html) will be removed in a future version of RabbitMQ: classic queues will remain a supported non-replicated queue type.

For cases that would benefit from replication and repeatable reads, [streams](https://www.rabbitmq.com/streams.html) may be a better option than quorum queues.
:::
<!-- markdownlint-enable DOCSMD004 -->
