
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
a standalone opc-proxy that can provide all supported connectors endpoint, :ref:`Kafka<Kafka-Connector>`, :ref:`InfluxDB`, :ref:`gRPC`.

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
        "grpcConnector" :   false,
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
    
    cd opcProxyConfigs
    # Env variable to make easier the next command
    OPC_LOCAL_CONF=$(pwd)

    docker create       \   # (1)
    --name proxy_test   \   # (2)
    --network="host"    \   # (3)
    -v ${OPC_LOCAL_CONF}:/app/configs  \  # (4)
    openscada/opc-proxy     # (5)

    # below the same command as above but in one line (copy-paste friendly)
    docker create --name proxy_test --network="host" -v ${OPC_LOCAL_CONF}:/app/configs openscada/opc-proxy

This is quite a long command, let's brake it and see what it means:

- It creates a container of the image in ``(5)`` named as defined in ``(2)``.
- In ``(3)`` set the ``localhost`` reference inside the container to point to the image host machine, 
  so one can use in the config file ``localhost`` to reference to a service running on the host machine. 
  If you would like to use the default docker networking option then you would need to find the IP of the docker ``network bridge``,
  more details in the Docker guide `Configure Networking <https://docs.docker.com/network/>`_.
- Line ``(4)`` is the most important, here we are mounting an external volume to the docker container, the syntax is simple: 
  ``-v absolute_path_to_host_dir : mirror_dir_in_container``, now all the content of the ``host_dir`` will be available to the docker 
  container dynamically. Here we want to pass the directory we just created that contains the configuration file. 

.. warning::
    the volume path must be an absolute path from the ``/``, even if the dir does not exist docker will not output an error.

.. tip::
    Docker containers must have different names, so unless you remove the container (`docker rm`) 
    you must change the name.

Run the Container
==================

First you need to start your OPC test server (see :ref:`OPC Test Server<OPC Test Server>`), then you can run the docker container:

.. code-block:: bash

    # start the container and attach output to STDIN, close with Ctrl-C
    docker start -i proxy_test

This should output something like this::
    
    2020-01-08 17:05:53.5762|INFO|OPCclient|Creating Application Configuration.
    2020-01-08 17:05:54.1004|WARN|OPCclient|Automatically accepting untrusted certificates. Do not use in production. Change in 'OPC.Ua.SampleClient.Config.xml'.
    2020-01-08 17:05:54.1004|INFO|OPCclient|Trying to connect to server endpoint:  opc.tcp://localhost:4840/freeopcua/server/
    2020-01-08 17:05:54.3017|INFO|OPCclient|Selected endpoint uses the following security policy: None
    2020-01-08 17:05:54.3017|INFO|OPCclient|Creating a session with OPC UA server.
    2020-01-08 17:05:54.3495|INFO|serviceManager|Loading nodes via browsing the OPC server...
    2020-01-08 17:05:54.3765|INFO|OPCclient|Surfing recursively trough server tree....
    2020-01-08 17:05:54.5011|INFO|cacheDB|Number of selected nodes: 1

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

    # Remove container
    docker rm __container_name__

    # remove image
    docker rmi  __image_name__



