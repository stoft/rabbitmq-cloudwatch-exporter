# RabbitMQ AWS CloudWatch exporter

A plugin for exporting the metrics collected by the RabbitMQ Management Plugin to [AWS CloudWatch](https://aws.amazon.com/cloudwatch/).

## Installing

Download the `.ez` files from the latest [release](https://github.com/noxdafox/rabbitmq-cloudwatch-exporter/releases) and copy them into the [RabbitMQ plugins directory](http://www.rabbitmq.com/relocate.html).

Enable the plugin:

```bash
[sudo] rabbitmq-plugins enable rabbitmq_cloudwatch_exporter
```

## Building from Source

Please see RabbitMQ Plugin Development guide.

To build the plugin:

```bash
git clone https://github.com/noxdafox/rabbitmq-cloudwatch-exporter.git
cd rabbitmq-cloudwatch-exporter
make dist
```

Then copy all the *.ez files inside the plugins folder to the [RabbitMQ plugins directory](http://www.rabbitmq.com/relocate.html) and enable the plugin:

```bash
[sudo] rabbitmq-plugins enable rabbitmq_cloudwatch_exporter
```

## Configuration

### AWS Credentials

To resolve AWS credentials, the standard environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are looked up first. Otherwise, the EC2 instance role or ECS task role are employed.

Alternatively, the User can specify them in the `rabbitmq.conf` file as follows.

```shell
cloudwatch_exporter.aws.access_key_id = "AKIAIOSFODNN7EXAMPLE"
cloudwatch_exporter.aws.secret_access_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

Or in the `rabbitmq.config` format.

```erlang
[{rabbitmq_cloudwatch_exporter,
  [{aws, [{access_key_id, "AKIAIOSFODNN7EXAMPLE"},
          {secret_access_key, "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"}]}]}].
```

### AWS Region

By default the metrics are published within the `us-east-1` AWS region. The region can be changed in the `rabbitmq.conf` file as follows.

```shell
cloudwatch_exporter.aws.region = "us-west-1"
```

Or in the `rabbitmq.config` format.

```erlang
[{rabbitmq_cloudwatch_exporter,
  [{aws, [{region, "us-west-1"}]}]}].
```

### Metrics collection

Metrics are grouped in different categories described below. Each category must be enabled in the configuration in order for its metrics to be exported.

`rabbitmq.conf` example.

```shell
cloudwatch_exporter.metrics.1 = overview
cloudwatch_exporter.metrics.2 = vhost
cloudwatch_exporter.metrics.3 = node
```

`rabbitmq.config` example.

```erlang
[{rabbitmq_cloudwatch_exporter,
  [{metrics, [overview, vhost, node, exchange, queue, connection, channel]}]}].
```

For `exchange`, `queue`, `connection` and `channel` groups, it is possible to control the names of the entities to be published via regular expressions.

`rabbitmq.conf` example.

```shell
# Only export exchanges which names begin with foo or end with bar
cloudwatch_exporter.export_regex.exchange = "foo.*|.*bar"
# Exclude server-named queues
cloudwatch_exporter.export_regex.queue = "^(?!amq.gen-.*).*$"
```

`rabbitmq.config` example.

```erlang
[{rabbitmq_cloudwatch_exporter,
  [{export_regex, [{exchange, "foo.*|.*bar"},
                   {queue, "^(?!amq.gen-.*).*$"}]}]}].
```

Metrics are exported every minute. The export period can be expressed in seconds via the `cloudwatch_exporter.export_period` configuration parameter (`rabbitmq_cloudwatch_exporter.export_period` in `rabbitmq.config` format).

Lastly, the [AWS CloudWatch namespace](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Namespace) can be controlled via the `cloudwatch_exporter.namespace` configuration parameter (`rabbitmq_cloudwatch_exporter.namespace` in `rabbitmq.config` format). The default value is `RabbitMQ`.

## Metrics

Metrics are grouped in different categories. All the metrics grouped within a category are characterised by a set of [dimensions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Dimension) which help to identify their belonging.

As AWS CloudWatch charges a [monthly cost](https://aws.amazon.com/cloudwatch/pricing/) per each metric, the CloudWatch Exporter Plugin exports a limited subset of metrics which are deemed relevant. If a particular metric is missing, please open an [Issue Ticket](https://github.com/noxdafox/rabbitmq-cloudwatch-exporter/issues) or submit a [Pull Request](https://github.com/noxdafox/rabbitmq-cloudwatch-exporter/pulls) to add it to the exported ones. For the same rationale, the CloudWatch Exporter Plugin only exports metrics from the categories which have been enabled in the configuration file.

### Overview

#### Dimensions

| Name        | Description       |
| ----------- | ----------------- |
| Metric      | "ClusterOverview" |
| Cluster     | Cluster name      |

#### Metrics

| Name                       | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| Messages                   | Total message count within the cluster                       |
| MessagesReady              | Messages ready for deliver within the cluster                |
| MessagesUnacknowledged     | Messages pending for acknowledgement within the cluster      |
| Ack                        | Messages acknowledged                                        |
| Confirm                    | Messages confirmed to publishers                             |
| Deliver                    | Messages delivered to consumers                              |
| DeliverGet                 | Messages delivered to consumers via basic.get                |
| DeliverNoAck               | Messages delivered to consumers without acknowledgment       |
| Get                        | Amount of basic.get requests                                 |
| GetEmpty                   | Amount of basic.get requests over empty queue                |
| GetNoAck                   | Amount of basic.get requests without acknowledgement         |
| Publish                    | Messages published within the channel                        |
| Redeliver                  | Messages redelivered                                         |
| ReturnUnroutable           | Messages returned as non routable                            |

### VHost

#### Dimensions

| Name        | Description       |
| ----------- | ----------------- |
| Metric      | "VHost"           |
| Cluster     | Cluster name      |
| VHost       | Virtual Host name |

#### Metrics

| Name                       | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| Messages                   | Total message count within the vhost                         |
| MessagesReady              | Messages ready for deliver within the vhost                  |
| MessagesUnacknowledged     | Messages pending for acknowledgement within the vhost        |
| Ack                        | Messages acknowledged                                        |
| Confirm                    | Messages confirmed to publishers                             |
| Deliver                    | Messages delivered to consumers                              |
| DeliverGet                 | Messages delivered to consumers via basic.get                |
| DeliverNoAck               | Messages delivered to consumers without acknowledgment       |
| Get                        | Amount of basic.get requests                                 |
| GetEmpty                   | Amount of basic.get requests over empty queue                |
| GetNoAck                   | Amount of basic.get requests without acknowledgement         |
| Publish                    | Messages published within the channel                        |
| Redeliver                  | Messages redelivered                                         |
| ReturnUnroutable           | Messages returned as non routable                            |

### Node

#### Dimensions

| Name        | Description              |
| ----------- | ------------------------ |
| Metric      | "Node"                   |
| Cluster     | Cluster name             |
| Node        | Node name                |
| Type        | Node type (disk\|memory) |
| Limit       | Limit when applicable    |

#### Metrics

| Name                    | Description                                  |
| ----------------------- | -------------------------------------------- |
| Uptime                  | Node uptime in milliseconds                  |
| Memory                  | Memory in use                                |
| DiskFree                | Amount of free disk                          |
| FileDescriptors         | Open file descriptors                        |
| Sockets                 | Open sockets                                 |
| Processes               | Erlang Processes                             |
| IORead                  | I/O read count                               |
| BytesIORead             | I/O read in bytes                            |
| IOWrite                 | I/O write count                              |
| BytesIOWrite            | I/O write in bytes                           |
| IOSeek                  | I/O seek count                               |
| MnesiaRamTransactions   | Mnesia transaction count on memory tables    |
| MnesiaDiskTransactions  | Mnesia transaction count on disk tables      |

### Exchange

#### Dimensions

| Name        | Description                                  |
| ----------- | -------------------------------------------- |
| Metric      | "Exchange"                                   |
| Cluster     | Cluster name                                 |
| Exchange    | Exchange name, `_` for the default exchange  |
| Type        | Exchange type (direct\|fanout\|...)          |
| VHost       | VHost where the exchange belongs             |

#### Metrics

| Name                    | Description                                  |
| ----------------------- | -------------------------------------------- |
| PublishIn               | Messages published within the exchange       |
| PublishOut              | Messages published by the exchange           |

### Queue

#### Dimensions

| Name        | Description                       |
| ----------- | --------------------------------- |
| Metric      | "Queue"                           |
| Cluster     | Cluster name                      |
| Queue       | Queue name                        |
| VHost       | VHost where the queue belongs     |

#### Metrics

| Name                       | Description                                                                      |
| -------------------------- | -------------------------------------------------------------------------------- |
| Memory                     | Memory consumed by the queue                                                     |
| Consumers                  | Consumers reading from the queue                                                 |
| Messages                   | Total message count within the queue                                             |
| MessagesReady              | Messages ready for deliver within the queue                                      |
| MessagesUnacknowledged     | Messages pending for acknowledgement within the queue                            |
| LengthPriorityLevel{level} | Number of elements in the priority level of the queue (only for priority queues) |
| Ack                        | Messages acknowledged                                                            |
| Deliver                    | Messages delivered to consumers                                                  |
| DeliverGet                 | Messages delivered to consumers via basic.get                                    |
| DeliverNoAck               | Messages delivered to consumers without acknowledgment                           |
| Get                        | Amount of basic.get requests                                                     |
| GetEmpty                   | Amount of basic.get requests over empty queue                                    |
| GetNoAck                   | Amount of basic.get requests without acknowledgement                             |
| Publish                    | Messages published within the queue                                              |
| Redeliver                  | Messages redelivered                                                             |

### Connection

#### Dimensions

| Name                    | Description                                      |
| ----------------------- | ------------------------------------------------ |
| Metric                  | "Connection"                                     |
| Cluster                 | Cluster name                                     |
| Connection              | IP:Port -> IP:port                               |
| Protocol                | Used protocol (AMQP 0-9-1\|AMQP 1.0\|STOMP\|...) |
| Node                    | Node where the connection is attached to         |
| VHost                   | VHost where the connection is attached to        |
| User                    | Username used to connect                         |
| AuthMechanism           | Employed authentication mechanism                |

#### Metrics

| Name                    | Description                                   |
| ----------------------- | --------------------------------------------- |
| Channels                | Opened channels count                         |
| Sent                    | Packets sent                                  |
| BytesSent               | Bytes sent                                    |
| Received                | Packets received                              |
| BytesReceived           | Bytes received                                |

### Channel

#### Dimensions

| Name                    | Description                                      |
| ----------------------- | ------------------------------------------------ |
| Metric                  | "Channel"                                        |
| Cluster                 | Cluster name                                     |
| Connection              | IP:Port -> IP:port                               |
| Channel                 | IP:Port -> IP:port (channels on the connection)  |
| VHost                   | VHost where the connection is attached to        |
| User                    | Username used to connect                         |

#### Metrics

| Name                       | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| MessagesUnacknowledged     | Messages pending acknowledgement on the channel              |
| MessagesUnconfirmed        | Messages unconfirmed on the channel                          |
| MessagesUncommitted        | Messages uncommitted on the channel                          |
| AknogwledgesUncommitted    | Acknowledgements uncommitted on the channel                  |
| PrefetchCount              | Prefetch count (QoS) configured on the channel               |
| GlobalPrefetchCount        | Global prefetch count (QoS) configured on the channel        |
| Ack                        | Messages acknowledged                                        |
| Confirm                    | Messages confirmed to publishers                             |
| Deliver                    | Messages delivered to consumers                              |
| DeliverGet                 | Messages delivered to consumers via basic.get                |
| DeliverNoAck               | Messages delivered to consumers without acknowledgment       |
| Get                        | Amount of basic.get requests                                 |
| GetEmpty                   | Amount of basic.get requests over empty queue                |
| GetNoAck                   | Amount of basic.get requests without acknowledgement         |
| Publish                    | Messages published within the channel                        |
| Redeliver                  | Messages redelivered                                         |
| ReturnUnroutable           | Messages returned as non routable                            |
