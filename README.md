# emqx_plugin_kafka

Kafka plugin for EMQX >= V5.4.0 && EMQX < V5.7.0

## Usage

### Release

```shell
> git clone https://github.com/jostar-y/emqx_plugin_kafka.git
> cd emqx_plugin_kafka
> make rel
_build/default/emqx_plugrel/emqx_plugin_kafka-<vsn>.tar.gz
```

### Config

#### Explain

```shell
> cat priv/emqx_plugin_kafka.hocon
plugin_kafka {
  // required
  connection {
    // Kafka client id: "emqx_plugin:kafka_client:${client_id}"
    // optional   default:"emqx_plugin:kafka_client:emqx_plugin_kafka_connection"
    client_id = "kafka_client"
    // Kafka address.
    // required
    bootstrap_hosts = ["10.3.64.223:9192", "10.3.64.223:9292", "10.3.64.223:9392"]

    // Reference type: kpro_connection:config().
    // https://github.com/kafka4beam/kafka_protocol/blob/master/src/kpro_connection.erl
    // optional   default:5s
    connect_timeout = 5s
    // enum: per_partition | per_broker
    // optional   default:per_partition
    connection_strategy = per_partition
    // optional   default:5s
    min_metadata_refresh_interval = 5s
    // optional   default:true
    query_api_versions = true
    // optional   default:3s
    request_timeout = 3s
    sasl {
      // enum:  plain | scram_sha_256 | scram_sha_512
      mechanism = plain
      username = "username"
      password = "password"
    }
    ssl {
      enable = false
    }

    //Emqx resource opts.
    // optional   default:32s
    health_check_interval = 32s
  }

  // optional
  producer {
    // Most number of bytes to collect into a produce request.
    // optional   default:896KB
    max_batch_bytes = 896KB
    // enum:  no_compression | snappy | gzip
    // optional   default:no_compression
    compression = no_compression
    // enum:  random | roundrobin | first_key_dispatch
    // optional   default:random
    partition_strategy = random

    // Encode kafka value.
    // enum:  plain | base64
    // optional   default:plain
    encode_payload_type = plain
  }

  // required
  hooks = [
    {
      // Hook point.
      // required
      endpoint = message.publish
      // Emqx topic pattern.
      // 1. Cannot match the system message;
      // 2. Cannot use filters that start with '+' or '#'.
      // message required
      filter = "test/#"
      // Kafka topic, must be created in advance in Kafka.
      // required
      kafka_topic = emqx_test
      // Matching template, value = ${.} indicates that all keys match
      // optional default:{timestamp = "${.timestamp}", value = "${.}",key = "${.clientid}"}
      kafka_message = {
        timestamp = "${.timestamp}"
        value = "${.}"
        key = "${.clientid}"
      }
    }
  ]
}
```

Some examples in the directory `priv/example/`.

#### Hook Point

|          endpoint           |  filter  |
| :-------------------------: | :------: |
|       client.connect        |    /     |
|       client.connack        |    /     |
|      client.connected       |    /     |
|     client.disconnected     |    /     |
|     client.authenticate     |    /     |
|      client.authorize       |    /     |
|     client.authenticate     |    /     |
| client.check_authz_complete |    /     |
|       session.created       |    /     |
|     session.subscribed      |    /     |
|    session.unsubscribed     |    /     |
|       session.resumed       |    /     |
|      session.discarded      |    /     |
|      session.takenover      |    /     |
|     session.terminated      |    /     |
|       message.publish       | required |
|      message.delivered      | required |
|        message.acked        | required |
|       message.dropped       | required |

#### Path

- Default path： `emqx/etc/emqx_plugin_kafka.hocon`
- Attach to path:  set system environment variables  `export EMQX_PLUGIN_KAFKA_CONF="absolute_path"`

## Attention
[scalability-and-performance](https://docs.emqx.com/en/emqx/v5.7/getting-started/feature-comparison.html#scalability-and-performance)
![image](https://github.com/user-attachments/assets/5304cba5-63ce-4b39-971d-a5559b3154c7)

The scalability and performance of open source versions after 5.6 have been limited, so the plug-in is no longer updated to support versions after 5.6
