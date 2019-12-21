:github_url: https://github.com/opc-proxy/opc-proxy-core

About the OPC-Proxy
^^^^^^^^^^^^^^^^^^^

The OPC-Proxy is a set of libraries aimed to facilitate communication 
between the industial autonomation world and **big data** tools.
It is a gateway to put in comunication a network/server that speaks
the OPC protocol with a network that supports a variety of other protocols.
The supported protocols/services to date are: HTTP, Kafka and InfluxDB.

This Proxy was developed to allow full duplex comunication between an
OPC-server and any of its clients, one of the main features is that it
support an RPC-style type of communication, so that to allow reads and
writes to and from an OPC-server.

This tool uses quite a few opensource libraries, we are gratefull to the `opc-foundation <https://github.com/OPCFoundation/UA-.NETStandard>`_,
the `lite-DB <https://www.litedb.org/>`_, the `Confluent-platform <https://www.confluent.io/>`_, the `gRPC framework <https://grpc.io/>`_,
whitout which it would have been impossible to reach this result.

Table of Contents
=================
.. toctree::
    :maxdepth: 2

    intro
    quickstart
    configuration
    connectors
    write_your_connector


