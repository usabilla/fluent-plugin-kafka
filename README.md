# fluent-plugin-kafka, a plugin for [Fluentd](http://fluentd.org)

[![Build Status](https://travis-ci.org/fluent/fluent-plugin-kafka.svg?branch=master)](https://travis-ci.org/fluent/fluent-plugin-kafka)

A fluentd plugin to both consume and produce data for Apache Kafka.

TODO: Also, I need to write tests

## Installation

Add this line to your application's Gemfile:

    gem 'fluent-plugin-kafka'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install fluent-plugin-kafka --no-document

If you want to use zookeeper related parameters, you also need to install zookeeper gem. zookeeper gem includes native extension, so development tools are needed, e.g. ruby-devel, gcc, make and etc.

## Requirements

- Ruby 2.1 or later
- Input plugins work with kafka v0.9 or later
- Output plugins work with kafka v0.8 or later

## Usage

### Common parameters

#### SSL authentication

- ssl_ca_cert
- ssl_client_cert
- ssl_client_cert_key
- ssl_ca_certs_from_system

Set path to SSL related files. See [Encryption and Authentication using SSL](https://github.com/zendesk/ruby-kafka#encryption-and-authentication-using-ssl) for more detail.

#### SASL authentication 

##### with GSSAPI

- principal
- keytab

Set principal and path to keytab for SASL/GSSAPI authentication. 
See [Authentication using SASL](https://github.com/zendesk/ruby-kafka#authentication-using-sasl) for more details.

##### with Plain/SCRAM

- username
- password
- scram_mechanism
- sasl_over_ssl

Set username, password, scram_mechanism and sasl_over_ssl for SASL/Plain or Scram authentication. 
See [Authentication using SASL](https://github.com/zendesk/ruby-kafka#authentication-using-sasl) for more details.

### Input plugin (@type 'kafka')

Consume events by single consumer.

    <source>
      @type kafka

      brokers <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,..
      topics <listening topics(separate with comma',')>
      format <input text type (text|json|ltsv|msgpack)> :default => json
      message_key <key (Optional, for text format only, default is message)>
      add_prefix <tag prefix (Optional)>
      add_suffix <tag suffix (Optional)>

      # Optionally, you can manage topic offset by using zookeeper
      offset_zookeeper    <zookeer node list (<zookeeper1_host>:<zookeeper1_port>,<zookeeper2_host>:<zookeeper2_port>,..)>
      offset_zk_root_node <offset path in zookeeper> default => '/fluent-plugin-kafka'

      # ruby-kafka consumer options
      max_bytes     (integer) :default => nil (Use default of ruby-kafka)
      max_wait_time (integer) :default => nil (Use default of ruby-kafka)
      min_bytes     (integer) :default => nil (Use default of ruby-kafka)
    </source>

Supports a start of processing from the assigned offset for specific topics.

    <source>
      @type kafka

      brokers <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,..
      format <input text type (text|json|ltsv|msgpack)>
      <topic>
        topic     <listening topic>
        partition <listening partition: default=0>
        offset    <listening start offset: default=-1>
      </topic>
      <topic>
        topic     <listening topic>
        partition <listening partition: default=0>
        offset    <listening start offset: default=-1>
      </topic>
    </source>

See also [ruby-kafka README](https://github.com/zendesk/ruby-kafka#consuming-messages-from-kafka) for more detailed documentation about ruby-kafka.

Consuming topic name is used for event tag. So when the target topic name is `app_event`, the tag is `app_event`. If you want to modify tag, use `add_prefix` or `add_suffix` parameters. With `add_prefix kafka`, the tag is `kafka.app_event`.

### Input plugin (@type 'kafka_group', supports kafka group)

Consume events by kafka consumer group features..

    <source>
      @type kafka_group

      brokers <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,..
      consumer_group <consumer group name, must set>
      topics <listening topics(separate with comma',')>
      format <input text type (text|json|ltsv|msgpack)> :default => json
      message_key <key (Optional, for text format only, default is message)>
      add_prefix <tag prefix (Optional)>
      add_suffix <tag suffix (Optional)>
      retry_emit_limit <Wait retry_emit_limit x 1s when BuffereQueueLimitError happens. The default is nil and it means waiting until BufferQueueLimitError is resolved>
      use_record_time <If true, replace event time with contents of 'time' field of fetched record>
      time_format <string (Optional when use_record_time is used)>

      # ruby-kafka consumer options
      max_bytes               (integer) :default => 1048576
      max_wait_time           (integer) :default => nil (Use default of ruby-kafka)
      min_bytes               (integer) :default => nil (Use default of ruby-kafka)
      offset_commit_interval  (integer) :default => nil (Use default of ruby-kafka)
      offset_commit_threshold (integer) :default => nil (Use default of ruby-kafka)
      fetcher_max_queue_size  (integer) :default => nil (Use default of ruby-kafka)
      start_from_beginning    (bool)    :default => true
    </source>

See also [ruby-kafka README](https://github.com/zendesk/ruby-kafka#consuming-messages-from-kafka) for more detailed documentation about ruby-kafka options.

Consuming topic name is used for event tag. So when the target topic name is `app_event`, the tag is `app_event`. If you want to modify tag, use `add_prefix` or `add_suffix` parameter. With `add_prefix kafka`, the tag is `kafka.app_event`.

### Output plugin

This plugin is for fluentd v1.0 or later. This will be `out_kafka` plugin in the future.

    <match app.**>
      @type kafka2

      brokers               <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,.. # Set brokers directly
      topic_key             (string) :default => 'topic'
      partition_key         (string) :default => 'partition'
      partition_key_key     (string) :default => 'partition_key'
      message_key_key       (string) :default => 'message_key'
      default_topic         (string) :default => nil
      default_partition_key (string) :default => nil
      default_message_key   (string) :default => nil
      exclude_topic_key     (bool)   :default => false
      exclude_partition_key (bool)   :default => false
      get_kafka_client_log  (bool)   :default => false
      headers               (hash)   :default => {}
      headers_from_record   (hash)   :default => {}
      use_default_for_unknown_topic (bool) :default => false

      <format>
        @type (json|ltsv|msgpack|attr:<record name>|<formatter name>) :default => json
      </format>

      # Optional. See https://docs.fluentd.org/v/1.0/configuration/inject-section
      <inject>
        tag_key tag
        time_key time
      </inject>

      # See fluentd document for buffer related parameters: https://docs.fluentd.org/v/1.0/configuration/buffer-section
      # Buffer chunk key should be same with topic_key. If value is not found in the record, default_topic is used.
      <buffer topic>
        flush_interval 10s
      </buffer>

      # ruby-kafka producer options
      idempotent        (bool)    :default => false
      sasl_over_ssl     (bool)    :default => false
      max_send_retries  (integer) :default => 1
      required_acks     (integer) :default => -1
      ack_timeout       (integer) :default => nil (Use default of ruby-kafka)
      compression_codec (string)  :default => nil (No compression. Depends on ruby-kafka: https://github.com/zendesk/ruby-kafka#compression)
    </match>

The `<formatter name>` in `<format>` uses fluentd's formatter plugins. See [formatter article](https://docs.fluentd.org/v/1.0/formatter).

ruby-kafka sometimes returns `Kafka::DeliveryFailed` error without good information.
In this case, `get_kafka_client_log` is useful for identifying the error cause.
ruby-kafka's log is routed to fluentd log so you can see ruby-kafka's log in fluentd logs.

Supports following ruby-kafka's producer options.

- max_send_retries - default: 1 - Number of times to retry sending of messages to a leader.
- required_acks - default: -1 - The number of acks required per request. If you need flush performance, set lower value, e.g. 1, 2.
- ack_timeout - default: nil - How long the producer waits for acks. The unit is seconds.
- compression_codec - default: nil - The codec the producer uses to compress messages.
- kafka_agg_max_bytes - default: 4096 - Maximum value of total message size to be included in one batch transmission.
- kafka_agg_max_messages - default: nil - Maximum number of messages to include in one batch transmission.
- max_send_limit_bytes - default: nil - Max byte size to send message to avoid MessageSizeTooLarge. For example, if you set 1000000(message.max.bytes in kafka), Message more than 1000000 byes will be dropped.
- discard_kafka_delivery_failed - default: false - discard the record where [Kafka::DeliveryFailed](http://www.rubydoc.info/gems/ruby-kafka/Kafka/DeliveryFailed) occurred
- monitoring_list - default: [] - library to be used to monitor. statsd and datadog are supported

If you want to know about detail of monitoring, see also https://github.com/zendesk/ruby-kafka#monitoring

See also [Kafka::Client](http://www.rubydoc.info/gems/ruby-kafka/Kafka/Client) for more detailed documentation about ruby-kafka.

This plugin supports compression codec "snappy" also.
Install snappy module before you use snappy compression.

    $ gem install snappy --no-document

snappy gem uses native extension, so you need to install several packages before.
On Ubuntu, need development packages and snappy library.

    $ sudo apt-get install build-essential autoconf automake libtool libsnappy-dev

On CentOS 7 installation is also necessary.

    $ sudo yum install gcc autoconf automake libtool snappy-devel

This plugin supports compression codec "lz4" also.
Install extlz4 module before you use lz4 compression.

    $ gem install extlz4 --no-document

This plugin supports compression codec "zstd" also.
Install zstd-ruby module before you use zstd compression.

    $ gem install zstd-ruby --no-document

#### Load balancing

Messages will be assigned a partition at random as default by ruby-kafka, but messages with the same partition key will always be assigned to the same partition by setting `default_partition_key` in config file.
If key name `partition_key_key` exists in a message, this plugin set the value of partition_key_key as key.

|default_partition_key|partition_key_key| behavior |
| --- | --- | --- |
|Not set|Not exists| All messages are assigned a partition at random |
|Set| Not exists| All messages are assigned to the specific partition |
|Not set| Exists | Messages which have partition_key_key record are assigned to the specific partition, others are assigned a partition at random |
|Set| Exists | Messages which have partition_key_key record are assigned to the specific partition with partition_key_key, others are assigned to the specific partition with default_parition_key |

If key name `message_key_key` exists in a message, this plugin publishes the value of message_key_key to kafka and can be read by consumers. Same message key will be assigned to all messages by setting `default_message_key` in config file. If message_key_key exists and if partition_key_key is not set explicitly, messsage_key_key will be used for partitioning.

#### Headers
It is possible to set headers on Kafka messages. This only works for kafka2 and rdkafka2 output plugin.

The format is like key1:value1,key2:value2. For example:

    <match app.**>
      @type kafka2
      [...]
      headers some_header_name:some_header_value
    <match>

You may set header values based on a value of a fluentd record field. For example, imagine a fluentd record like:

    {"source": { "ip": "127.0.0.1" }, "payload": "hello world" }

And the following fluentd config:

    <match app.**>
      @type kafka2
      [...]
      headers_from_record source_ip:$.source.ip
    <match>

The Kafka message will have a header of source_ip=12.7.0.0.1.

The configuration format is jsonpath. It is descibed in https://docs.fluentd.org/plugin-helper-overview/api-plugin-helper-record_accessor

### Buffered output plugin

This plugin uses ruby-kafka producer for writing data. This plugin works with recent kafka versions. This plugin is for v0.12. If you use v1, see `kafka2`.

    <match app.**>
      @type kafka_buffered

      # Brokers: you can choose either brokers or zookeeper. If you are not familiar with zookeeper, use brokers parameters.
      brokers             <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,.. # Set brokers directly
      zookeeper           <zookeeper_host>:<zookeeper_port> # Set brokers via Zookeeper
      zookeeper_path      <broker path in zookeeper> :default => /brokers/ids # Set path in zookeeper for kafka

      topic_key             (string) :default => 'topic'
      partition_key         (string) :default => 'partition'
      partition_key_key     (string) :default => 'partition_key'
      message_key_key       (string) :default => 'message_key'
      default_topic         (string) :default => nil
      default_partition_key (string) :default => nil
      default_message_key   (string) :default => nil
      output_data_type      (json|ltsv|msgpack|attr:<record name>|<formatter name>) :default => json
      output_include_tag    (bool) :default => false
      output_include_time   (bool) :default => false
      exclude_topic_key     (bool) :default => false
      exclude_partition_key (bool) :default => false
      get_kafka_client_log  (bool) :default => false

      # See fluentd document for buffer related parameters: https://docs.fluentd.org/v/0.12/buffer

      # ruby-kafka producer options
      idempotent                   (bool)    :default => false
      sasl_over_ssl                (bool)    :default => false
      max_send_retries             (integer) :default => 1
      required_acks                (integer) :default => -1
      ack_timeout                  (integer) :default => nil (Use default of ruby-kafka)
      compression_codec            (string)  :default => nil (No compression. Depends on ruby-kafka: https://github.com/zendesk/ruby-kafka#compression)
      kafka_agg_max_bytes          (integer) :default => 4096
      kafka_agg_max_messages       (integer) :default => nil (No limit)
      max_send_limit_bytes         (integer) :default => nil (No drop)
      discard_kafka_delivery_failed   (bool) :default => false (No discard)
      monitoring_list              (array)   :default => []
    </match>

### Non-buffered output plugin

This plugin uses ruby-kafka producer for writing data. For performance and reliability concerns, use `kafka_bufferd` output instead. This is mainly for testing.

    <match app.**>
      @type kafka

      # Brokers: you can choose either brokers or zookeeper.
      brokers        <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,.. # Set brokers directly
      zookeeper      <zookeeper_host>:<zookeeper_port> # Set brokers via Zookeeper
      zookeeper_path <broker path in zookeeper> :default => /brokers/ids # Set path in zookeeper for kafka

      default_topic         (string) :default => nil
      default_partition_key (string) :default => nil
      default_message_key   (string) :default => nil
      output_data_type      (json|ltsv|msgpack|attr:<record name>|<formatter name>) :default => json
      output_include_tag    (bool) :default => false
      output_include_time   (bool) :default => false
      exclude_topic_key     (bool) :default => false
      exclude_partition_key (bool) :default => false

      # ruby-kafka producer options
      max_send_retries    (integer) :default => 1
      required_acks       (integer) :default => -1
      ack_timeout         (integer) :default => nil (Use default of ruby-kafka)
      compression_codec   (string)  :default => nil (No compression. Depends on ruby-kafka: https://github.com/zendesk/ruby-kafka#compression)
      max_buffer_size     (integer) :default => nil (Use default of ruby-kafka)
      max_buffer_bytesize (integer) :default => nil (Use default of ruby-kafka)
    </match>

This plugin also supports ruby-kafka related parameters. See Buffered output plugin section.

### rdkafka based output plugin

This plugin uses `rdkafka` instead of `ruby-kafka` for ruby client.
You need to install rdkafka gem.

    # rdkafka is C extension library so need development tools like ruby-devel, gcc and etc
    $ gem install rdkafka --no-document

`rdkafka2` is for fluentd v1.0 or later.

    <match app.**>
      @type rdkafka2

      brokers <broker1_host>:<broker1_port>,<broker2_host>:<broker2_port>,.. # Set brokers directly

      topic_key             (string) :default => 'topic'
      default_topic         (string) :default => nil
      partition_key         (string) :default => 'partition'
      partition_key_key     (string) :default => 'partition_key'
      message_key_key       (string) :default => 'message_key'
      default_topic         (string) :default => nil
      default_partition_key (string) :default => nil
      default_message_key   (string) :default => nil
      exclude_topic_key     (bool) :default => false
      exclude_partition_key (bool) :default => false
      headers               (hash)   :default => {}
      headers_from_record   (hash)   :default => {}

      <format>
        @type (json|ltsv|msgpack|attr:<record name>|<formatter name>) :default => json
      </format>

      # Optional. See https://docs.fluentd.org/v/1.0/configuration/inject-section
      <inject>
        tag_key tag
        time_key time
      </inject>

      # See fluentd document for buffer section parameters: https://docs.fluentd.org/v/1.0/configuration/buffer-section
      # Buffer chunk key should be same with topic_key. If value is not found in the record, default_topic is used.
      <buffer topic>
        flush_interval 10s
      </buffer>

      # You can set any rdkafka configuration via this parameter: https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md
      rdkafka_options {
        "log_level" : 7
      }
    </match>

If you use v0.12, use `rdkafka` instead.

    <match kafka.**>
      @type rdkafka

      default_topic kafka
      flush_interval 1s
      output_data_type json

      rdkafka_options {
        "log_level" : 7
      }
    </match>

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
