.. _remotecontrol:

Remote control
##############

The **Open Ephys HTTP Server** enables remote control of the GUI via a RESTful API. Immediately upon launching, the GUI starts an HTTP server on port **37497** (:code:`EPHYS` on a phone keypad) of the machine running the GUI. You can confirm that the server is running by pointing a web browser to :code:`http://localhost:37497/api/processors`. This should display a JSON string with information about the current signal chain.

The HTTP server can be disabled or re-enabled via the **File** menu.

The following sections document the various remote control commands that are available. All examples are in Python, based on the `requests <https://requests.readthedocs.io/en/latest/>`__ library.

Start/stop acquisition and recording
------------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/status`"

Querying the GUI's acquisition/recording status uses an HTTP :code:`GET` request at the :code:`/api/status` endpoint:

.. code-block:: Python

    r = requests.get("http://localhost:37497/api/status")

This returns a JSON string (accessible via :code:`r.json()`) containing information about the current :code:`mode` of the GUI, which can take the following values:

* :code:`IDLE` - the GUI is not acquiring data
* :code:`ACQUIRE` - the GUI is acquiring data, but not recording
* :code:`RECORD` - the GUI is acquiring and recording data

To set the GUI's status, use an HTTP :code:`PUT` request:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/status",
        json={"mode" : "ACQUIRE"})

.. note:: The signal chain must contain at least one Record Node in order for the :code:`RECORD` command to work.

Get/set recording configuration
-------------------------------
        
.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/recording`"

Information related to recording can be queried and updated using the :code:`/api/recording` endpoint. Sending a :code:`GET` request to this endpoint will return a JSON string with information about the recording directory as well as the details of each record node:

.. code-block:: Python

    r = requests.get("http://localhost:37497/api/recording")

:code:`r.json()` will return a JSON string with the following structure:

.. code-block:: js

    {
        'parent_directory' : '/Users/neuroscientist/Documents/OpenEphys',
        'base_text' : 'AUTO',
        'prepend_text' : 'NONE', 
        'append_text' : 'AUTO', 
        'record_nodes' : [
            {
                'node_id' : 102,
                'parent_directory' : '/Users/neuroscientist/Documents/OpenEphys',
                'record_engine' : 'BINARY',
                'experiment_number' : 1,
                'recording_number' : 3,
                'is_synchronized' : true
            }, ...
        ]
    }

To update the default location for storing data, use an HTTP :code:`PUT` request to set the :code:`parent_directory` field:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/recording",
        json={"parent_directory" : "/Users/neuroscientist/Documents/Data"})

Note that this new directory will be applied only to future Record Nodes, not Record Nodes that are already present in the signal chain.

To change the recording directory for a specific Record Node, the Record Node's ID must be appended to the address, e.g.:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/recording/102",
        json={"parent_directory" : "/Users/neuroscientist/Documents/Data"})

To use custom base text for the next recording directory (in place of the auto-generated date string), use the following command:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/recording",
        json={"base_text" : "new_directory_name"})

The same endpoint can be used to set the recording directory :code:`prepend_text` and :code:`append_text` as well.


Get information about the signal chain
---------------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/processors`"

Sending a :code:`GET` request to the :code:`/api/processors` endpoint will return a JSON string with information about available processors and their parameters. Information about Record Nodes is accessed separately, via the :code:`/api/recording` endpoint.

.. code-block:: Python

    r = requests.get("http://localhost:37497/api/processors")

will return a string with the following structure:

.. code-block:: js

    {
        "processors" : [
            { 
              "id" : 100,
              "name" : "File Reader", 
              "parameters" : [ ],
              "predecessor" : null,
              "streams" : [
                { 
                  "channel_count" : 16,
                  "name" : "example_data", 
                  "parameters" : [ ],
                  "sample_rate" : 40000.0,
                  "source_id" : 100
                }
              ]
            },
            {
              "id" : 101,
              "name" : "Bandpass Filter",
              "parameters" : [ ],
              "predecessor" : 100, 
              "streams": [ 
                {
                  "channel_count" : 16,
                  "name" : "example_data", 
                  "sample_rate" : 40000.0,
                  "source_id" : 100
                  "parameters" : [
                    {
                      "name" : "enable_stream",
                      "type" : "Boolean",
                      "value" : "true"
                    },
                    {
                      "name" : "high_cut",
                      "type" : "Float",
                      "value" : "6000"
                    },
                    {
                      "name" : "low_cut",
                      "type" : "Float",
                      "value" : "300"},
                    {
                      "name" : "Channels",
                      "type" : "Mask Channels",
                      "value" : ""
                    }
                    ],
                  }
                ]
            }
    }

Appending the 3-digit processor ID to the endpoint (e.g., :code:`/api/processors/101`) will return information about one processor at a time.

Send a configuration message to a specific processor
----------------------------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/processors/<processor_id>/config`"

Certain processors can respond to custom configuration messages that modify their state prior to starting acquisition. For example, the following command will change the reference setting on a Neuropixels probe connected to slot 3, port 1, and dock 1:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/processors/100/config",
        json={"text" : "NP REFERENCE 3 1 1 TIP"})


Broadcast a message to all processors 
-------------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/message`"

Broadcast messages are relayed to all processors while acquisition is active. These messages will be ignored unless a plugin has implemented the :code:`handleBroadcastMessage()` method, and knows how to respond to the specific message that was sent. For example, the following command will trigger a 100 ms pulse on digital output line 1 of the Open Ephys Acquisition Board:

.. code-block:: js

    r = requests.put(
        "http://localhost:37497/api/message",
        json={"text" : "ACQBOARD TRIGGER 1 100"})

.. tip:: Broadcast messages are saved by all Record Nodes, so these messages can be used to mark different epochs within a given recording.

Close the GUI remotely
-------------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/window`"

To shut down the GUI, send the **quit** command to the :code:`/api/window` endpoint:

.. code-block:: Python

    r = requests.put(
        "http://localhost:37497/api/window",
        json={"command" : "quit"})
