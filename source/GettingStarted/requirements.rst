Requirements
=============


This library is written in C#, so you would need to install Microsoft .Net Core framework on your machine.
This step is not necessary in case you use the provided Docker image of the standalone application.
You do need it if you want to write a custom OPC-Proxy, in which case you would need to build the library 
into an executable.

.. note::
    There are two ways to get up and running: using the docker image or compiling the code. 
    A `standalone OPC-Proxy <https://github.com/opc-proxy/opcProxy-Standalone>`_ is provided in both cases, 
    this has already the basic functionality and just need to be configured.

In case you want to build your own custom application you would need the following:

- .NET Core >= 2.2  following the description `here <https://dotnet.microsoft.com/download>`_
- Your favourite OPC test server
- Git

OPC Test Server
================

This library is an OPC-client which is quite useless without an OPC-server to connect to.
It is very usefull to have a test OPC-server that you can configure and run to quicly check 
and play arount with the OPC-Proxy library.

There are many opensource (and proprietary) library to write your own server, 
we reccomend two, that are decently complete and easy to use:

- The Python library (this is the default for all the examples)
- The NodeJS library

Setup an OPC-Server with Python
""""""""""""""""""""""""""""""""

The Python based OPCUA library is maybe not the most complete but certainly one of the 
simplest to get up and running. Install the module:

``pip install opcua``

Copy and paste the server example from `this repository <https://github.com/FreeOpcUa/python-opcua/blob/master/examples/server-minimal.py>`_ into a file,
call it for example ``server-minimal.py``, for completeness we report the file below:

.. code-block:: python

    # server-minimal.py

    import sys
    sys.path.insert(0, "..")
    import time


    from opcua import ua, Server


    if __name__ == "__main__":

        # setup our server
        server = Server()
        server.set_endpoint("opc.tcp://0.0.0.0:4840/freeopcua/server/")

        # setup our own namespace, not really necessary but should as spec
        uri = "http://examples.freeopcua.github.io"
        idx = server.register_namespace(uri)

        # get Objects node, this is where we should put our nodes
        objects = server.get_objects_node()

        # populating our address space
        myobj = objects.add_object(idx, "MyObject")
        myvar = myobj.add_variable(idx, "MyVariable", 6.7)
        myvar.set_writable()    # Set MyVariable to be writable by clients

        # starting!
        server.start()
        
        try:
            count = 0
            while True:
                time.sleep(1)
                count += 0.1
                myvar.set_value(count)
        finally:
            #close connection, remove subcsriptions, etc
            server.stop()


Run it:

.. code-block:: sh
    
    python server-minimal.py

This runs an OPC-server on port ``4840`` that expose an ever increasing variable of type Double called ``MyVariable``.
Now you need to run the OPC-Proxy to listen for that variable changes.


Setup the NodeJS OPC-Server
"""""""""""""""""""""""""""

If you are familiar with NodeJS, a quite good library to check out is the `Node-OPCUA <http://node-opcua.github.io/>`_.
For this you need to install `NodeJS <https://nodejs.org/en/>`_ and `NPM <https://www.npmjs.com/>`_, once you have done it is quite simple:

.. code-block:: sh

    git clone git@github.com:node-opcua/node-opcua-sampleserver.git
    cd node-opcua-sampleserver
    npm install
    node server.js

This will start a server on ``port: 26543`` and will expose two variables, one called ``Temperature`` and the other ``MyVariable2``.