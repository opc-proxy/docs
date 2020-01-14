Configuration
=============

Configuration can be done via JSON file, the default file name is ``proxy_config.json``.
All the config keys that are not recognized will be ignored, if no configuration is provided
for a parameter its default value will be loaded. An example of configuration file can be the following:

.. code-block:: javascript

    {
        "opcServerURL":"opc.tcp://localhost:4840/freeopcua/server/",

        "loggerConfig" :{
            "loglevel" :"info"
        },

        "nodesLoader" : {
            "targetIdentifier" : "browseName",
            "whiteList":["MyVariable"]
        },
        "gRPC" :{
                "port" : 5051
        }
    }


OPC related Configs
"""""""""""""""""""
These configs are related to core features and define how the opc client must behave. They must be  placed at the root level of the json file.

.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**opcServerURL**", "string","none", "OPC server TCP URL endpoint"
    "**reconnectPeriod**","int", "10 [s]", "Time interval [seconds] to wait before retry to reconnect to OPC server"
    "**publishingInterval**", "int", "1000 [ms]", "This is a subscription parameter, time intervall [millisecond] at which the OPC server will send node values updates."
    "**opcSystemName**", "string","OPC", "Name of the OPC system that will be used for identification "

Nodes Loader 
^^^^^^^^^^^^^
These are config related to nodes loading methods and selection rules.

.. note::
    If no selection rules are defined then **ALL** nodes of type **Variable** are loaded automatically.
    If more selection rules are defined they Add/Excludes nodes (of type Variable) based on the following priority order: ``whiteList``,
    ``blackList``, ``notContain``, ``contains``, ``matchRegEx``.

.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**browseNodes**", "bool", "true", "If ``true`` load nodes via recursively drilling trough the server tree, it may use many network requests. If false will load nodes from an xml file, according to the Nodeset2 OPC specification."
    "**targetIdentifier**", "string", "DisplayName", "Node attribute that undergoes selection rules, it can be: ``displayname``, ``browsename``, ``nodeid``. In case of ``nodeid`` is necessary to specify also the prefix like ``i=123`` or ``s=VarName``. It is case insensitive."
    "**filename**", "string", "nodeset.xml", "Path to the xml file where the nodes are defined. Necessary if ``browseNodes = false``."
    "**whiteList**", "string[ ]", "empty", "Accept all nodes with ``targetIdentifier`` **exactly equal** to one of the string in the list. Runs first."
    "**blackList**", "string[ ]", "empty", "Exclude nodes with ``targetIdentifier`` **exactly equal** to one of the string in the list. Runs second."
    "**notContain**",  "string[ ]", "empty", "Excludes nodes with  ``targetIdentifier`` **containing** one of the string in the list. Runs before ``contains`` and ``matchRegEx``."
    "**contains**", "string[ ]", "empty", "Accept nodes with  ``targetIdentifier`` **containing** one of the string in the list. Runs before ``matchRegEx``."
    "**matchRegEx**",  "string[ ]", "empty", "Accept nodes with  ``targetIdentifier`` **matching** one of the regular expression string in the list. Runs last."


gRPC-Connector Configs
""""""""""""""""""""""
These configs are related to the :ref:`gRPC Connector`, they must be placed under the key ``gRPC`` as follows:

.. code-block:: javascript

    {
        // Other config here 

        "gRPC" :{
                "port" : 5051
        }
    }


.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**host**", "string","localhost", "host name on the network."
    "**port**", "int","5051", "Port on which to listen for client requests"


Kafka-Connector Configs
"""""""""""""""""""""""
These configs are related to the :ref:`Kafka-Connector`, they must be placed under the keys ``kafkaProducer`` and ``kafkaRPC``,
there are also two root level configs: ``KafkaSchemaRegistryURL`` and ``KafkaServers``, as in the example:

.. code-block:: javascript

    {
        // Other config here 

        opcSystemName : "OPC",
        KafkaSchemaRegistryURL : "localhost:8081",
        KafkaServers : "localhost:9092",
        
        kafkaProducer : {
            // Producer conf
        },
        kafkaRPC : {
            // RPC conf
        }
    }

Root level cofigs:
^^^^^^^^^^^^^^^^^^^^

.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**opcSystemName**","string","OPC","System name is a core variable, it will be used to evaluate the topic names for nodes publishing, see :ref:`Kafka-Connector`"
    "**KafkaSchemaRegistryURL**","string","localhost:8081","Endpoint of the schema registry"
    "**KafkaServers**","string","localhost:9092","Comma separated list of kafka brokers. These will be set for the producer and the consumer of the OPC-Proxy, this can be overidden, see below."

kafkaProducer:
^^^^^^^^^^^^^^^^^

.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**BootstrapServers**", "string", "localhost:9092", "Comma separated list of Kafka brokers endpoints. If not set, this will be set to the value of **KafkaServers**."
    "**BatchNumMessages**", "int", "10000", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**LingerMs**", "int", "100 [ms]", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**QueueBufferingMaxKbytes**", "int", "1048576 [Kbytes]", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**QueueBufferingMaxMessages**", "int", "100000", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**MessageTimeoutMs**", "int", "300000", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**EnableIdempotence**", "bool", "false", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**RetryBackoffMs**", "int", "100 [ms]", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"
    "**MessageSendMaxRetries**", "int", "2", "See `Confluent producer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ProducerConfig.html#Confluent_Kafka_ProducerConfig_BatchNumMessages>`_"

kafkaRPC:
^^^^^^^^^^^^^^^^^
All the non reported kafka consumer configurations are set to default values.

.. csv-table::
    :header: "Config Key","type","Default","Notes"
    :widths: 20, 10, 10, 40

    "**BootstrapServers**", "string", "localhost:9092", "Comma separated list of Kafka brokers endpoints. If not set, this will be set to the value of **KafkaServers**."
    "**GroupId**", "string", "OPC", "Group ID of the RPC kafka consumer. No other consumer can have this group ID in the whole system. If not set, default is to be set to ``opcSystemName``."
    "**enableKafkaRPC**", "bool", "true", "Enable the RPC-style comunication trough kafka topics."
    "**EnableAutoCommit**", "bool", "true", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**EnableAutoOffsetStore**", "bool", "true", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**AutoCommitIntervalMs**", "int", "5000 [ms]", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**SessionTimeoutMs**", "int", "10000 [ms]", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**AutoOffsetReset**", "string", "latest", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**EnablePartitionEof**", "bool", "false", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**FetchWaitMaxMs**", "int", "1000 [ms]", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**FetchMinBytes**", "int", "1", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"
    "**HeartbeatIntervalMs**", "int", "3000 [ms]", "See `Confluent consumer docs <https://docs.confluent.io/current/clients/confluent-kafka-dotnet/api/Confluent.Kafka.ConsumerConfig.html#Confluent_Kafka_ConsumerConfig_AutoCommitIntervalMs>`_"



InfluxDB-Connector Configs
""""""""""""""""""""""""""
