
==============================
Run With Docker
==============================

Docker is a way to distribute self-contained applications easily. 
We provide a Docker image for the Community Edition that you can very easily 
install and upgrade on your servers. Your machine needs to have the **Docker Engine  Communiti Edition (CE)** installed 
first. Refer to the docker `installation page <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_

Download
==========

.. code-block:: console

    docker pull openscada/opc-proxy

This will pull an image with the .NET framework dependencies already installed and with a compiled executable.
The image is built from `this repository <https://github.com/opc-proxy/opcProxy-Standalone>`_, it contains
a standalone opc-proxy that can provide all supported connectors endpoint, :ref:`Kafka`, :ref:`InfluxDB`, :ref:`gRPC`.

Configure
===========

Create a configuration file:
""""""""""""""""""""""""""""""

.. code-block:: sh

    # host directory to share with the docker container
    mkdir  opcProxyConfigs
    # the configuration file must be called "proxy_config.json"
    touch opcProxyConfigs/proxy_config.json

Edit the config:
""""""""""""""""

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

        },
        "httpConnector" :   false,
        "influxConnector" : false,
        "kafkaConnector":   false
    }

This will tell the OPC-Proxy that:

- Needs to connect to an OPC server at the specified URL, we use a python test server as described in :ref:`Setup an OPC-Server with Python`, 
  if you are using another test server you need to update that line.
- The nodesLoader here will match against a whitelist all nodes of the server, it will look for a Node with ``BrowseName`` attribute (see :ref:`OPC Data Structure`) 
  equals to  ``MyVariable``, which is default for our test server.
- The log level is set to ``DEBUG``, so that we will see the output of the variable changing.
- All connectors are set to ``false``, meaning that this proxy will only connect to the opc-server and nothing more.

Setup a docker container:
"""""""""""""""""""""""""

.. code-block:: bash

    docker create       \   # (1)
    --name proxy_test   \   # (2)
    --network="host"    \   # (3)
    -v absolute_path_to_config_dir:/app/configs  \  # (4)
    openscada/opc-proxy     # (5)

    # below the same command as above but in one line (copy-paste friendly)
    docker create --name proxy_test --network="host" -v absolute_path_to_config_dir:/app/configs openscada/opc-proxy

This is quite a long command, let's brake it and see what it means:

- It creates a container of the image in ``(5)`` named as defined in ``(2)``.
- In ``(3)`` set the ``localhost`` reference inside the container to point to the image host machine, 
  so one can use in the config file ``localhost`` to reference to a service running on the host machine. 
  If you would like to use the default docker networking option then you would need to find the IP of the docker ``network bridge``,
  more details in the Docker guide `Configure Networking <https://docs.docker.com/network/>`_.
- Line ``(4)`` is the most important, here we are mounting an external volume to the docker container, the syntax is simple: 
  ``absolute_path_to_host_dir``:``mirror_dir_in_container``, now all the content of the ``host_dir`` will be available to the docker 
  container dynamically. Here we want to pass the directory we just created that contains the configuration file. 

.. warning::
    the volume path must be an absolute path from the ``/``, even if the dir does not exist docker will not output an error.

.. tip::
    Docker containers must have different names, so unless you remove the container (`docker rm`) 
    you must change the name.

Run the Container
==================

First you need to start your OPC test server (see `above <>`_), then you can run the docker container:

.. code-block:: bash

    # start the container and attach output to STDIN, close with Ctrl-C
    docker start -i proxy_test

This should output something like this::

    2019-12-22 23:37:23.6756|INFO|cacheDB|Creating Application Configuration.
    2019-12-22 23:37:24.0252|WARN|cacheDB|Automatically accepting untrusted certificates. Do not use in production. Change in 'OPC.Ua.SampleClient.Config.xml'.
    2019-12-22 23:37:24.0263|INFO|cacheDB|Discover endpoints of opc.tcp://localhost:4334/UA/MyLittleServer.
    2019-12-22 23:37:24.3415|INFO|cacheDB|    Selected endpoint uses: Basic128Rsa15
    2019-12-22 23:37:24.3415|INFO|cacheDB|Creating a session with OPC UA server.
    Accepted Certificate: CN=NodeOPCUA-TEST, O=NodeOPCUA, L=Paris, S=IDF, C=FR
    2019-12-22 23:37:24.4921|INFO|serviceManager|Loading nodes via browsing the OPC server...
    2019-12-22 23:37:24.5379|INFO|cacheDB|Surfing recursively trough server tree....
    2019-12-22 23:37:24.5379|DEBUG|cacheDB| DisplayName, BrowseName, NodeClass
    2019-12-22 23:37:24.6746|DEBUG|cacheDB|Retriving data types of the selected nodes...
    2019-12-22 23:37:24.6746|DEBUG|cacheDB|Adding Node MyVariable1  of type System.Double
    2019-12-22 23:37:24.6916|WARN|cacheDB|number of nodes : 1
    2019-12-22 23:37:24.6938|INFO|cacheDB|Creating a subscription with publishing interval of 1 second.
    2019-12-22 23:37:24.6938|INFO|cacheDB|Adding a list of monitored nodes to the subscription.
    2019-12-22 23:37:24.6938|INFO|cacheDB|Number of nodes to be monitored: 1
    2019-12-22 23:37:24.7040|INFO|cacheDB|Adding the subscription to the session.
    2019-12-22 23:37:24.7642|INFO|serviceManager|Running...Press Ctrl-C to exit...
    2019-12-22 23:37:25.7711|DEBUG|cacheDB|value -> 300.0463184435  type --> System.Double
    2019-12-22 23:37:25.7799|DEBUG|cacheDB|Updating value for MyVariable1 to 300.0463184435 at 12/22/2019 22:04:01
    2019-12-22 23:37:26.7137|DEBUG|cacheDB|value -> 332.340097954784  type --> System.Double

Usefull Docker Commands
"""""""""""""""""""""""""

.. code-block:: bash

    # start container in the background
    docker start proxy_test

    # stop container
    docker stop proxy_test
    
    # restart container (usefull when edit config)
    docker restart proxy_test

    # list running container
    docker ps

    # list all containers
    docker ps -a 



