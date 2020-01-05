=======================
Build Your Proxy 
=======================

In case you want to add your own feature to the proxy or not familiar with docker,
here we explain all the steps needed to write and compile to executable your own proxy.

We are going to build from scratch a proxy very similar to the one in the `standalone <https://github.com/opc-proxy/opcProxy-Standalone>`_
repository.

Install .NET
===============

If you havn't done it yet, and especially if you are using a Linux machine. 
You can find instruction on how to do it on the `Microsoft Website <https://dotnet.microsoft.com/download>`_.
You will need:

- .NET Core >= 3.1
- We suggest to install all three library: **.NET core SDK, .NET core Runtime, ASP .NET core runtime**.



Start with a clone Project
===========================

Probably the easiest is to clone the standalone repository and to take it as starting point. 

.. code-block:: console

    git clone git@github.com:opc-proxy/opcProxy-Standalone.git
    cd opcProxy-Standalone/

Now build it:

.. code-block:: console
    
    dotnet build

Now you can jump to `Add a Configuration file`_ and continue from there.


Create a project from scratch
=============================

Here we describe in steps how to create a .NET project that can be compiled to executable
and runs a basic opc-proxy. You can get the same result by cloning the standalone repository 
described in `Start with a clone Project`_ section.

**Crate a new blank project:**

.. code-block:: console

    dotnet new console --name myproxy
    cd myproxy

This is a scaffolding command, which created a directory with a file **Program.cs** and a configuration file **myproxy.csproj**.
Now you can compile this "hello world" program and run it like this:

.. code-block:: console

    dotnet build
    dotnet run

The console should output ``Hello World!``

Install OPC-Proxy libs
"""""""""""""""""""""""

The library can be installed using the package manager of .NET, `nuget <https://www.nuget.org/>`_. In this demo we will use 
all the suppoted packages, of course you can install only the one you need, the following command will install their lastest 
version:

.. code-block:: console

    dotnet add package opcProxy.core
    dotnet add package opcProxy.GrpcConnector
    dotnet add package opcProxy.InfluxDBConnector
    dotnet add package opcProxy.KafkaConnector 


Modify the Program
"""""""""""""""""""

Open the file ``Program.cs`` and edit it like below:

.. code-block:: c#

    using System;
    using OpcProxyCore;

    namespace myproxy
    {
        class Program
        {
    
            static int Main(string[] args)
            {
                // instantiaing the manager, 
                // this will load configuration from file or args
                serviceManager manager = new serviceManager(args);
                
                // This runs the OPC-Proxy manager with all core 
                // functionalities: connects to server, monitor items...
                manager.run();
                return 0; 
            }
            
        }
    }

Now build with ``dotnet build``, there should be no error.

Add a Configuration file
"""""""""""""""""""""""""

Configuration can only be given via ``JSON`` file format. A configuration file is necessary, the 
program will stop otherwise. The default config file name is ``proxy_config.json`` and the program look 
for it in the directory where you run it. You can change the path or the name of the config file via the 
``--config path_to_file`` flag at run time.

Create the following file (it is already there in case you are cloning the repo) in the main directory and name it ``proxy_config.json``:


.. code-block:: js

    /* proxy_config.json */
    {
        "opcServerURL":"opc.tcp://localhost:4840/freeopcua/server/",

        "loggerConfig" :{
            "loglevel" :"debug"
        },
        
        "nodesLoader" : {
            "targetIdentifier" : "browseName", 
            "whiteList":["MyVariable"]

        }
    }

This will tell the OPC-Proxy that:

- Needs to connect to an OPC server at the specified URL, we use a python test server as described in :ref:`Setup an OPC-Server with Python`, 
  if you are using another test server you need to update that line.
- The nodesLoader here will match against a whitelist all nodes of the server, it will look for a Node with ``BrowseName`` attribute (see :ref:`OPC Data Structure`) 
  equals to  ``MyVariable``, which is default for our test server.
- The log level is set to ``DEBUG``, so that we will see the output of the variable changing.

There are many configuration options and possibilities for loading nodes, they are described in detail in the :ref:`Configuration` section.


Run The Proxy
=============

Before actually running the program we need two things:

- An additional config file. Copy `this file <https://github.com/opc-proxy/opcProxy-Standalone/blob/master/Opc.Ua.SampleClient.Config.xml>`_ from the 
  standalone repository and place it in the main directory (not needed if you cloned it). The file name is important, so keep same naming ``Opc.Ua.SampleClient.Config.xml``.
  Soon this will not be needed anymore, refer to `issue #17 <https://github.com/opc-proxy/opc-proxy-core/issues/17>`_.
- Run your favourite test opc-server. Remember the configuration we used will only work for the :ref:`Python test server<Setup an OPC-Server with Python>`.

Now you can simply do:

.. code-block:: bash

    dotnet run
    # Or for a custom config file
    dotnet run --config __path_to_file__

Press ``Ctrl-C`` to end the process.


Adding Connectors
"""""""""""""""""

Up to now the OPC-Proxy would only connect to the opc-server, browse its variable tree and subscribe
to change of the variables that match the ``nodeLoader`` criteria. Now we will add **connectors** that 
will allow Read, Write, Subscribe acess to the external world. 

If you followed the `Start with a clone Project`_ section you can add connectors via config file, by adding the following 
options:

.. code-block:: js

    {
        /* some config.... */


        "httpConnector" :   false,
       "influxConnector" : false,
       "kafkaConnector":   false
    }

Turnig ``true``/``false`` those switches you can enable/disable the corresponding connector.
You can find more details on each of these connectors and their configurations in the :ref:`Connectors` section.


If you followed `Create a project from scratch`_ instead, then to add connectors you need to modify the ``Program.cs``. 
Any connector must implement the *OPC-Proxy interface* so independently of its implementation you can add any connector as follows:

.. code-block:: c#

    // load the library at the beginning of the file
    using OpcGrpcConnect;
    using OpcInfluxConnect;
    using opcKafkaConnect;

            .
            .
            .
    // Initialize and add the connectors to the serviceManager
    // before the call to "manager.run();"
    
    HttpImpl opcHttpConnector = new HttpImpl();
    manager.addConnector(opcHttpConnector);
    
    InfluxImpl influx = new InfluxImpl();
    manager.addConnector(influx);

    KafkaConnect kafka = new KafkaConnect();
    manager.addConnector(kafka);

You can find more details on each of these connectors and their configurations in the :ref:`Connectors` section.

.. note::
    The ``InfluxDB`` and ``Kafka`` connectors will not work if the respective servers are not running.






