===========
Connectors
===========

The OPC-proxy can connect with only one OPC-server, on the other hand it can put 
the OPC-server in comunication with a large number of clients, each of these 
clients can use its own favourite protocol. The OPC-proxy has a modular design, 
to add new capabilities one simply needs to add the corresponding connector. 
Connectors are modules for the OPC-proxy that implement an endpoint for a communication protocol,
they can leverage the OPC-proxy core library to interact with the OPC-server. 
To write your own connector see the :ref:`Extend Connectors` section.
In this section we describe the currenlty supported connectors: `gRPC <gRPC Connector>`_, `Kafka <Kafka-Connector>`_ and `InfluxDB`_.

An OPC-Proxy **can run multiple connectors**, so for example one can push metrics to InfluxDB while
serving a gRPC or Kafka (or both) endpoint.

gRPC Connector
==============

gRPC is a modern open source high performance RPC framework, initially 
developed at Google. It is very flexible and userfriendly, it can easily 
put in communication different services independently on the programming 
language used. For more information visit `grpc.io <https://grpc.io/>`_.

As a comunication layer gRPC uses HTTP2, while it uses 
`protocol buffers <https://developers.google.com/protocol-buffers/>`_
as serialization/deserialization and Interface Definition Language.

The OPC-Proxy gRPC-Connector repository can be found at `gitHub-GrpcConnector <https://github.com/opc-proxy/GrpcConnector>`_.
You find all its configuration options in the :ref:`Configuration` section.


Protocol Definition
"""""""""""""""""""

A client can initiate ``Read`` request and a ``Write`` request. 
In future also a subscritpion to server push on variable change will be available.
The read/write request and response are defined in the proto-config-file that you can download 
from this `gitHub-repo <https://github.com/opc-proxy/GrpcConnector/blob/master/opcGrpcConnect/opc.grpc.connect.proto>`_,
you can use the proto-config to generate automatically the code needed for the comunication in almost
any language. 

A node is represented as:

.. code-block:: typescript

    NodeValue : {
        name : string,   // name of the node
        value : string,  // is always a string you need to deserialize it
        timestamp : string, // format ISO 8601 - UTC
        type : string      // int, bool, float, string 
    }

The value of the nodes is always serialized to ``string``, is you responsability to deserialize it
to the ``type`` specified.

A read request and its response is defined as:

.. code-block:: typescript

    ReadOpcNodes( ReadRequest ) returns ( ReadResponse )
    // where:
    ReadRequest : string[];  // list of node names to be read

    ReadResponse : {
        nodes : NodeValue[], // list of read nodes
        isError : bool,
        errorMessage : string
    }

The data exchanged in a write request is:

.. code-block:: typescript

    WriteRequest : {
        name : string,
        value : string
    }

    WriteResponse : {
        isError : bool,
        errorMessage : string
    } 

The value of the write request need to be serialized always to a ``string``, the OPC-Proxy knows what is 
the right type and will take care of the conversion. In case of conversion error or OPC-server error it will 
return an error message (error message still in preparation see `issue #1 <https://github.com/opc-proxy/GrpcConnector/issues/1>`_)in the response and the ``isError`` boolean will be true. A successful write request
will have a ``isError = false``.



gRPC Client Example
"""""""""""""""""""
An example client based on NodeJs is provided in the `gitHub-OPC-Node-Client-Example <https://github.com/opc-proxy/OPC-Node-Client-Examples/tree/master/Examples/gRPC>`_
Follow the instruction reported there.

First run the OPC-Proxy configured with a gRPC endpont, this example assumes an OPC-Proxy running on ``port:5051``, 
which is default, it also assume that the OPC-server is the `Python-OPCUA <https://github.com/FreeOpcUa/python-opcua>`_, 
or in general that there will be an exposed variable called ``MyVariable``.  

The example will read and write a value to ``MyVariable`` of the python test server example.
The value of MyVariable is always increasing by 0.1 every half a second. The client will read
its value and reset it to 1. 

Keep in mind that the OPC-server will push variables values (if they change) to the OPC-Proxy
with rate of 1 sec, you can query the OPC-Proxy much faster than that, the write request will be forwared
to the server immediately, but read request will read the latest value from the memory cache of the
OPC-Proxy.


Kafka-Connector
===============

`Apache Kafka <https://kafka.apache.org/>`_ is an open-source stream-processing platform,
it is the de facto standard for high-throughput, low-latency handling of real-time data feeds.

The Kafka-Connector add the ability to the opc-proxy to stream data to a kafka server. It supports:

- Sending a message on a topic when a node value changes (notification form opc-server)
- Bidirectional comunication, ``read/write`` and possibly more, with the PLC using an RPC protocol. The protocol supported is `JSON-RPC-2.0 <https://www.jsonrpc.org/specification>`_.

This library uses the `Avro <https://avro.apache.org/>`_ serialization library, which allows great flexibility in defining the structure of the data 
exchanged. As storage engine for data schemas we are using the `Confluent SchemaRegistry <https://www.confluent.io/confluent-schema-registry/>`_, which is necessary for this library.
In the future a ``JSON`` based serialization option will be available and so the additional complexity of a schema registry will not be 
required anymore (see `issue #4 <https://github.com/opc-proxy/KafkaConnectorLib/issues/4>`_).

The OPC-Proxy Kafka-connector repository can be found at `gitHub-KafkaConnectorLib <https://github.com/opc-proxy/KafkaConnectorLib>`_.
You find all its configuration options in the :ref:`Configuration` section.

Data Streams
""""""""""""
The Kafka-Connector will by default define three **topics** the name of which depends on the configuration variable ``opcSystemName``: 

- The **Metrics-topic**, the one containing a stream of nodes value on change, is named as the ``opcSystemName`` configuration variable.
- The **RPC-request** topic, the one where all the (write) request are send, is named ``opcSystemName`` with appended suffix ``-request``.
- The **RPC-response** topic, the one where all the RPC-style responses are served, is named ``opcSystemName`` with appended suffix ``-response``.

.. note::
    Keep in mind that your consumer clients need to have different ``group ID`` if you want all of them to receive updates.
    Also **Do Not** assign same ``group ID`` as the the OPC-Proxy to any other clients.

Serialization Deserialization
"""""""""""""""""""""""""""""
In Kafka messages are organized as a key:value pair. ``Key`` and ``Value`` can be serialized/deserialized with independent 
serializers, we choose to serialize ``Keys`` using the standard string-serializer, so the message key will always be a strings, while the values are serialized using 
the `Avro <https://avro.apache.org/>`_ framework. 

In Avro one describes the data structure in a JSON-schema, the schema is stored in a schema-registry server, each message has an ID that refer to 
that schema, so each client can deserialize the messages dinamically. One can add a data type or modify an existing schema and the client will 
be able to properly deserialize the data without the need of additional code. So changing data structure (if backwords compatible) will not break
clients. It also give flexibility, one can configure it to send messages with different data content on the same topic and the 
consumer will always be able to deserialize it correctly. Here we use this last property, independently on the Node value type, 
the consumer will always get the value properly deserialized (and already with the right type) depending  on which language you are using.

For the **Metrics-topic**, the one where are streamed the nodes values on change, we use as Kafka **message Key** the node variable name, 
while as Kafka **message Value** we use the following Avro schema:

.. code-block:: typescript
    
    {
        type: 'record',
        name: datatype + 'Type',
        fields:[
            {
                name:'value', 
                type: datatype
            }
        ]
    }

Where ``datatype`` can be: ``string``, ``double``, ``float``, ``boolean``, ``int``, ``long``.


Kafka-RPC
"""""""""
For the RPC-style comunication we are using Kafka as a simple message broker, the default configuration of the producer and consumer 
of the RPC-topics are such that the comunication between the OPC-server and your client is preformed with latency of the order of ``10 ms``.
The protocol used used for this comunication is defined in the `JSON-RPC spec <https://www.jsonrpc.org/specification>`_.
Even tough Kafka might seems inadequate for an RPC-style comunication, we find that it makes communication in a system 
(with many microservices) simpler and more flexible, it is a way to standardize comms, allowing for example a ``kafka-stream`` or a 
Storm-bolt to write easily to an OPC-server.

You can find the Avro schema used for the **RPC-request**  and **RPC-response** topics 
at `gitHub-RPC-Schemas <https://github.com/opc-proxy/OPC-Node-Client-Examples/tree/master/Examples/Kafka/AvroSchemas>`_.
We tried to make it as close as possible to the original spec, but there are a few differences. 
The **RPC-request** kafka **message-key** is a string and is controlled by the client, the Kafka-Connector does nothing
with it except making sure that the corresponding **RPC-response** will have the same message-key.
A  request message will look like:

.. code-block:: typescript

    // Request
    {
        method : string,
        params : string[]  // list of strings
        id: long or null
    }

Where the only ``method`` supported by now is ``write`` (but in the future might be more) and ``params`` is expected to contain 
a list with two strings, the first one representing the name of the node, the second one its value. The Kafka-Connector will take care
of serializing the value from string to the correct data type expected by the OPC-server. The ``id`` if provided will be forwarded to 
the response, in case of ``null`` then the Kafka-Connector will return the Kafka offset as ``id`` in the response, in this way you can let 
Kafka worry to generate system-wide unique ids for you (well, in this case topic-wide unique ids), you can collect the offset at 
request time, tide it to the topic and store it memory, then wait for the corresponding response.

A response message value in the  **RPC-response** topics, would look like:

.. code-block:: typescript

    // Response
    {
        result : string or null,

        error : {  // != null only if an error exist
            code : integer,
            message : string, 
        } 

        id: long 
    }

Given that the only supported method is "write", the ``result`` will be a string representing the written node value to 
the OPC-server, and will differ from ``null`` only in case of successful write operation.
The ``error`` object will only be present in case of an error (when "result" is null).
The ``id`` is either forwarded from the request or the Kafka-Offset of the related request-message in the RPC-request topic.

.. note::
    If the RPC-request topic has more than one partition and the ``id`` is set to ``null`` in a request, the response ``id`` will be ambiguous.
    Please inform us if you have such a use case.


Kafka-Connector Client Example
""""""""""""""""""""""""""""""

You can find an example of kafka client for NodeJs in this repository `gitHub-NodeKafka_client <https://github.com/opc-proxy/OPC-Node-Client-Examples/tree/master/Examples/Kafka>`_.
Here we assume as opc test-server the :ref:`OPCUA-Python-server<Setup an OPC-Server with Python>`. 
Using Node we show how to connect to a published data stream of nodes change on kafka, and how to interface to the kafka-RPC for writing nodes values.

InfluxDB
=========

`InfluxDB <https://www.influxdata.com/>`_ is an open-source time series database
optimized for fast, high-availability storage and retrieval of time series data in fields 
such as operations monitoring, application metrics, Internet of Things sensor 
data, and real-time analytics.

- Library used
- pushing metrics


