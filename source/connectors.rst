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
In this section we describe the currenlty supported connectors: `gRPC`_, `Kafka`_ and `InfluxDB`_.

An OPC-Proxy **can run multiple connectors**, so for example one can push metrics to InfluxDB while
serving a gRPC or Kafka (or both) endpoint.

gRPC
=======

gRPC is a modern open source high performance RPC framework, initially 
developed at Google. It is very flexible and userfriendly, it can easily 
put in communication different services independently on the programming 
language used. For more information visit `grpc.io <https://grpc.io/>`_.

As a comunication layer gRPC uses HTTP2, while it uses 
`protocol buffers <https://developers.google.com/protocol-buffers/>`_
as serialization/deserialization and Interface Definition Language.

You find the configuration option for this connector package in the :ref:`Configuration` section.

HTTP Connector API
""""""""""""""""""

A client can initiate ``Read`` request and a ``Write`` request. 
In future also a subscritpion to server push on variable change will be available.
The read/write request and response are defined in the proto-config-file that you can download 
from the `repository <https://github.com/opc-proxy/GrpcConnector/blob/master/opcGrpcConnect/opc.grpc.connect.proto>`_,
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



HTTP Client Example
"""""""""""""""""""
An example client based on NodeJs is provided in this `repository <https://github.com/opc-proxy/OPC-Node-Client-Examples/tree/master/Examples/gRPC>`_
Follow the instruction reported there.

First run the OPC-Proxy configured with a gRPC endpont, this example assumes an OPC-Proxy running on ``port:5051``, 
which is default, it also assume that the OPC-server is the `Python-OPCUA <https://github.com/FreeOpcUa/python-opcua/blob/master/examples/server-minimal.py>`_, 
or in general that there will be an exposed variable called ``MyVariable``.  

The example with will read and write a value to ``MyVariable`` of the python test server example. 
Keep in mind that the OPC-server will push variables values (if they change) to the OPC-Proxy
with rate of 1 sec, you can query the OPC-Proxy much faster than that, the write request will be forwared
to the server immediately, but read request will read the latest value from the memory cache of the
OPC-Proxy.


Kafka
=====

`Apache Kafka <https://kafka.apache.org/>`_ is an open-source stream-processing platform,
it is the de facto standard for high-throughput, low-latency handling of real-time data feeds.

The Kafka-Connector add the ability to the opc-proxy to stream data to a kafka server. It supports:

- Sending a message on a topic when a node value changes (notification form opc-server)
- Bidirectional comunication, ``read/write`` and possibly more, with the PLC using an RPC protocol. The protocol supported is `JSON-RPC-2.0 <https://www.jsonrpc.org/specification>`_.

This library uses the `Avro <https://avro.apache.org/>`_ serialization library, which allows great flexibility in defining the structure of the data 
exchanged. As storage engine for data schemas we are using the `Confluent SchemaRegistry <https://www.confluent.io/confluent-schema-registry/>`_, which is necessary for this library.
In the future a ``JSON`` based serialization option will be available and so the additional complexity of a schema registry will not be 
required anymore (see `issue #4 <https://github.com/opc-proxy/KafkaConnectorLib/issues/4>`_).

Data Streams
""""""""""""
The Kafka-Connector will by default define three **topics** the name of which depends on the configuration variable ``opcSystemName``: 

- The data stream of nodes value change will be directed on topic named as the ``opcSystemName``.
- All the RPC-style requests need to be send to topic name ``opcSystemName``-request.
- All the RPC-style responses will be served in a topic named ``opcSystemName``-response.

For the RPC-style comunication we are using Kafka as a simple message broker, the default configuration of the producer and consumer 
of the RPC-topics are such that the comunication between the OPC-server and your client is preformed with latency of the order of ``10 ms``.

Serialization deserialization
"""""""""""""""""""""""""""""

Kafka-RPC
"""""""""

The protocol used used for this comunication is defined in the `JSON-RPC spec <https://www.jsonrpc.org/specification>`_, 
here we go quicly trough it by examples.


Example repository
""""""""""""""""""


InfluxDB
=========

`InfluxDB <https://www.influxdata.com/>`_ is an open-source time series database
optimized for fast, high-availability storage and retrieval of time series data in fields 
such as operations monitoring, application metrics, Internet of Things sensor 
data, and real-time analytics.

- Library used
- pushing metrics


