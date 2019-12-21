****************
Introduction
****************

What is OPC-Proxy?
==================

This is a library to build an OPC proxy. It allows to deploy an IoT gateway to connect any OPC server
with your network of microservices or cloud.
This library is suitable for monitoring and also controlling devices, while tipically 
other vendors of IoT gateways only offer monitoring.
We focused on defining a protocol for bidirectional communication exposing the user to
a simple API, so that one can read, but also write values to the OPC server without knowing details 
about OPC.

**Features:**
    - Suitable for **monitoring** and **controlling** devices.
    - Simple API.
    - Reliable OPC client build with the `OPC-foundation <https://github.com/OPCFoundation/UA-.NETStandard>`_ standard library.
    - Modular design with external connectors that can be added, extended and customized.
    - Supported connectors: **HTTP, Kafka, InfluxDB**.
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


Connectors 
==========

The API
========





