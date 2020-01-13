Configuration
===============

Configuration can be done via JSON file, the default file name is ``proxy_config.json``.
All the config keys that are not recognized will be ignored, if no configuration is provided
for a parameter its default value will be loaded. An example of configuration file can be the following:

.. code-block:: javascript

    {
        "opcServerURL":"opc.tcp://localhost:4840/freeopcua/server/",

        "loggerConfig" :{
            "loglevel" :"info"
        },

        "nodesLoader" : {
            "targetIdentifier" : "browseName",
            "whiteList":["MyVariable"]
        },
        "gRPC" :{
                "port" : 5051
        }
    }


OPC related Configs
"""""""""""""""""""
These configs are related to core features and define how the opc client must behave. They must be  placed at the root level of the json file.

.. csv-table::
    :header: "Config Key","Default","Notes"
    :widths: 20, 20, 40

    "opcServerURL", "none", "OPC server TCP URL endpoint"
    "reconnectPeriod", "10", "Time interval [seconds] to wait before retry to reconnect to OPC server"
    "publishingInterval", "1000", "This is a subscription parameter, time intervall [millisecond] at which the OPC server will send node values updates."
    "opcSystemName", "OPC", "Name of the OPC system that will be used to identification "


gRPC-Connector Configs
""""""""""""""""""""""



Kafka-Connector Configs
"""""""""""""""""""""""

InfluxDB-Connector Configs
""""""""""""""""""""""""""
