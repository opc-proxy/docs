****************
Introduction
****************


What is OPC-Proxy?
==================

The OPC-Proxy allows to build and deploy a customized IoT gateway to connect any OPC server
with your network of microservices or cloud.
This library is suitable for monitoring and control of devices, while tipically vendors of IoT gateways only offer monitoring.
We focused on defining a protocol for bidirectional communication exposing the user to
a simple API, so that one can read, but also write values to the OPC server without knowing details 
about OPC.

**Features:**
    - Suitable for **monitoring** and **controlling** devices.
    - Simple API.
    - Reliable OPC client build with the `OPC-foundation <https://github.com/OPCFoundation/UA-.NETStandard>`_ standard library.
    - Load nodes from an XML file (nodes2set) or simply browsing the server
    - Powerful Nodes loading selection options
    - Modular design with external connectors that can be added, extended and customized.
    - Supported connectors: :ref:`HTTP <gRPC>`, :ref:`Kafka`, :ref:`InfluxDB`.
    - Written in C#.


Basics of OPC 
=============

OPC is an opensource protocol used in industrial autonomation. It allows real time communication
beween rugged industrial devices. 
The full specification is more than a 1000 pages and is quite complex (you can find it `here <https://reference.opcfoundation.org/v104/>`_), it describes several use cases 
and different comunication options, here we aim to support OPC UA. In this section we try to summarize briefly the important notions that are needed in the following.

OPC Data Structure 
""""""""""""""""""""""
Data in an OPC server is organized in a tree structure, similar to a filesystem, there are *folders*, which contain *objects*, each object can contain *variables*.
Each variable has an associated *data type* and various *references*. Now, one can access any item of the tree, each item is generalized as a **Node**.
To access any node one needs to know its **node id**  and the **namespace** under which it is stored. The OPC-proxy can interact only with 
nodes related to variables. Each node that is related to a variable has the following properties:

.. admonition:: **Variable Node properties**
    :class: props
    
    :NodeID: examples are *"ns=3;s=var_name"* for string identifier and *"ns=3;i=1234"* for integer identifier, where *ns* is the namespace.
        Weather the identifier is a string or a number depends on the server implementation.
    :BrowseName: *Server given name*, it is usually related to the variable name, but it can contain additional caracters or can be truncated.
        It depends on the server implementation.
    :DisplayName: *Human readable* variable name, this is usually the name that is displayed to the user.
    :Value: Value of the variable.


Loading Nodes
"""""""""""""""
A client does not in general know the node ids of the variables on which wants to operate on, usually they are strings related to the variable name,
but in general these can be random integers, depending on the implementation of the server. There are two ways to make an opc-proxy aware of the node ids for
the relevant variables, one can feed an XML file produced from the server with all nodes using the *node2set* OPC specification. Or one can simply 
browse each branch of the tree for variable nodes using consequent network calls, this last option is referred to as *browsing*. 

.. WARNING:: **Browsing feature:**
    The use of browsing feature for a server that contains many variables may lead to a large amout of network requests and 
    can take long time (up to minutes). This of course depends on the server implementation.

Read, Write and Subscribe
""""""""""""""""""""""""""
An OPC client can *read*, *write* and *subscribe* to change of a server variable. 
Both read and write are single network calls, one can specify to read and write multiple variables at the same time.
The OPC-proxy only support to read and write *values* on variable nodes, does not support any other attribute.
Once the OPC-proxy is connected to the server and the session is open, one can monitor a list of variables for their change,
these are called *monitored items*, the server will push those change when they occur.

Behaviours
""""""""""""
:Session: Once the session with the opc-server is established, the OPC-proxy will the care of keeping the connection alive,
    in case of network interruption the OPC-proxy will keep trying to reconnect at user defined intervals.
:Node Publishing: once one subscribe for monitor a list of variables, the OPC-server will push their updated values and statuses at intervals
    to save network requests. The server will aggregate all the change for all the variables within a certain user defined time, 
    to minimize the network requests it will send all the changes within the interval in a single network call. The OPC-proxy gives the user the
    possibility to set this time intervall.
:Node Monitoring: The OPC-proxy will automatically monitor (subscribe to change) all nodes specified in the configuration file.
:Read and Write: The OPC-proxy will only allow to read and write the nodes specified in the configuration file.  
:Memory Cache: The OPC-proxy chaches the latest value for each variable in a in-memory database, so it will always return the value from 
    cache for each read request made by the client and will not hit the OPC-server.


Connectivity Modules 
=====================

The OPC-proxy can connect with only one OPC-server, on the other hand it can put the OPC-server in comunication with a large number of clients,
each of these clients can use its own favourite protocol. The OPC-proxy has a modular design, to add new capabilities one simply needs to 
add the corresponding connector. Connectors are modules for the OPC-proxy that implement an endpoint for a communication protocol,
they can leverage the OPC-proxy core library to interact with the OPC-server. To write your own connector see the :ref:`Extend Connectors` section.

The currently supported connectors are:
    - **gRPC:** Implements an RPC type of comunication between a server and a client over HTTP. It uses the gRPC framework, see more details in the :ref:`gRPC` connector section.
    - **Kafka:** Implements a data stream to a Kafka topic trught the *Kafka producer* library. Implements an RPC type of comunication trough Kafka topics using the JSON-RPC protocol, it accepts write requests. More details in the :ref:`Kafka` connector section.
    - **InfluxDB:** Submits a stream of metrics to InfulxDB on variables change. More details in the :ref:`InfluxDB` connector section.


