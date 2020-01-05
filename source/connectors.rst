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

API
"""

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

A read request an its response is defined as:

.. code-block:: typescript

    ReadOpcNodes( ReadRequest ) returns ( ReadResponse )
    // where:
    ReadRequest : string[];  // list of strings

    ReadResponse : {
        nodes : NodeValue[], // list of nodes
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
return an error message in the response.



Example of usage repository
""""""""""""""""""""""""""""
An example client with NodeJs is provided in this `repository <https://github.com/opc-proxy/OPC-Node-Client-Examples/tree/master/Examples/gRPC>`_
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

Data Streams
""""""""""""

Serialization deserialization
""""""""""""""""""""""""""""""

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


