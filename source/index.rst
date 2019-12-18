:github_url: https://github.com/opc-proxy/opc-proxy-core

Overview
========

The OPC-Proxy is a set of libraries aimed to facilitate communication
between the industial autonomation world and **big data** tools.
It is a gateway to put in comunication a network/server that speaks
the OPC protocol with a network that supports a variety of other protocols.
The supported protocols/services to date are: HTTP, Kafka and InfluxDB.

This Proxy was developed to allow full duplex comunication between an
OPC-server and any of its clients, one of the main features is that it
support an RPC-style type of communication, so that to allow reads and
writes to and from an OPC-server.

This tool uses quite a few opensource libraries whitout which it would have
been impossible to reach this result. We are gratefull to the opc-foundation,
the lite-DB, the Confluent-platform, for their hard work to make the internet
a better place.

.. 
    FIXME add links


Table of Contents
=================
.. toctree::
    :maxdepth: 2

    intro
    quickstart
    configuration
    connectors
    write_your_connector


