.. _remotecontrol:

Remote control
##############

The **Open Ephys HTTP Server** enables remote control of the GUI via an HTTP API. Immediately upon launching, the GUI starts a server on port **37497** (:code:`EPHYS` on a phone keypad). You can confirm that the server is running by opening :code:`http://localhost:37497/api/processors` in a browser. If you are using a different computer, replace :code:`localhost` with the IP address of the machine running the GUI.

The HTTP server can be disabled or re-enabled via the **File** menu.

Most read-only endpoints use HTTP :code:`GET`, while endpoints that change GUI state use :code:`PUT` with a JSON body. Python examples below use the `requests <https://requests.readthedocs.io/en/latest/>`__ library, while Matlab examples use `webread <https://www.mathworks.com/help/matlab/ref/webread.html>`__ and `webwrite <https://www.mathworks.com/help/matlab/ref/webwrite.html>`__.

Malformed JSON requests return HTTP 400 responses. Requests for missing processors, streams, or parameters return HTTP 404 responses.

Quick API reference
-------------------

.. csv-table::
   :header: "Method", "Endpoint", "Description"
   :widths: 8, 46, 46

   "GET", ":code:`/api/status`", "Return the current GUI mode."
   "PUT", ":code:`/api/status`", "Set the GUI mode to :code:`IDLE`, :code:`ACQUIRE`, or :code:`RECORD`."
   "GET", ":code:`/api/recording`", "Return default recording settings and all Record Node settings."
   "PUT", ":code:`/api/recording`", "Update default recording settings."
   "PUT", ":code:`/api/recording/<record_node_id>`", "Update a specific Record Node."
   "GET", ":code:`/api/config`", "Return the current signal-chain configuration as XML wrapped in JSON."
   "PUT", ":code:`/api/load`", "Load a signal chain from disk."
   "PUT", ":code:`/api/save`", "Save the current signal chain to disk."
   "GET", ":code:`/api/processors/list`", "List processor types that can be added to the signal chain."
   "GET", ":code:`/api/processors`", "Return all processors currently in the signal chain."
   "GET", ":code:`/api/processors/<processor_id>`", "Return one processor and its streams."
   "GET", ":code:`/api/processors/<processor_id>/parameters`", "Return all processor-level parameters."
   "GET", ":code:`/api/processors/<processor_id>/parameters/<parameter_name>`", "Return one processor-level parameter."
   "GET", ":code:`/api/processors/<processor_id>/streams/<stream_index>`", "Return one stream and its parameters."
   "GET", ":code:`/api/processors/<processor_id>/streams/<stream_index>/parameters`", "Return all parameters for one stream."
   "GET", ":code:`/api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>`", "Return one stream parameter."
   "PUT", ":code:`/api/processors/<processor_id>/parameters/<parameter_name>`", "Set one processor-level parameter."
   "PUT", ":code:`/api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>`", "Set one stream parameter."
   "PUT", ":code:`/api/processors/<processor_id>/config`", "Send a processor-specific configuration message."
   "PUT", ":code:`/api/message`", "Broadcast a message to all processors."
   "GET", ":code:`/api/processors/clear`", "Clear the signal chain."
   "PUT", ":code:`/api/processors/add`", "Add a processor to the signal chain."
   "PUT", ":code:`/api/processors/delete`", "Delete a processor from the signal chain."
   "GET", ":code:`/api/undo`", "Undo the previous action."
   "GET", ":code:`/api/redo`", "Redo the previous action."
   "GET", ":code:`/api/cpu`", "Return the current audio callback CPU usage."
   "GET", ":code:`/api/latency`", "Return processor latency information for each stream."
   "GET", ":code:`/api/audio/devices`", "List available audio device types and device names."
   "GET", ":code:`/api/audio/device`", "Return the currently selected audio device and supported rates and buffer sizes."
   "PUT", ":code:`/api/audio`", "Change the active audio device, sample rate, or buffer size."
   "PUT", ":code:`/api/quit`", "Close the GUI."

Query and control acquisition state
-----------------------------------

Use :code:`GET /api/status` to query the GUI mode and :code:`PUT /api/status` to change it.

.. code-block:: python

        import requests

        status = requests.get("http://localhost:37497/api/status").json()
        requests.put(
                "http://localhost:37497/api/status",
                json={"mode": "ACQUIRE"},
        )

.. code-block:: matlab

        status = webread('http://localhost:37497/api/status');

        out = webwrite(
                'http://localhost:37497/api/status',
                struct('mode','ACQUIRE'),
                weboptions('RequestMethod','put','MediaType','application/json'));

The returned JSON contains a single :code:`mode` field:

* :code:`IDLE` means the GUI is not acquiring data.
* :code:`ACQUIRE` means the GUI is acquiring but not recording.
* :code:`RECORD` means the GUI is both acquiring and recording.

.. note:: The signal chain must contain at least one Record Node in order for :code:`RECORD` mode to succeed.

Recording configuration
-----------------------

Use :code:`GET /api/recording` to inspect the global recording configuration and the state of each Record Node.

.. code-block:: python

        recording = requests.get("http://localhost:37497/api/recording").json()

.. code-block:: json

        {
                "parent_directory": "/Users/neuroscientist/Documents/OpenEphys",
                "base_text": "AUTO",
                "prepend_text": "NONE",
                "append_text": "AUTO",
                "default_record_engine": "BINARY",
                "record_nodes": [
                        {
                                "node_id": 102,
                                "parent_directory": "/Users/neuroscientist/Documents/OpenEphys",
                                "record_engine": "BINARY",
                                "experiment_number": 1,
                                "recording_number": 3,
                                "is_synchronized": true
                        }
                ]
        }

Use :code:`PUT /api/recording` to update global defaults. Supported fields are:

* :code:`parent_directory`
* :code:`prepend_text`
* :code:`base_text`
* :code:`append_text`
* :code:`default_record_engine`
* :code:`start_new_directory`

Example:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/recording",
                json={
                        "parent_directory": "/Users/neuroscientist/Documents/Data",
                        "base_text": "experiment_01",
                        "append_text": "mouse_a",
                        "default_record_engine": "BINARY",
                        "start_new_directory": "true",
                },
        )

.. code-block:: matlab

        out = webwrite(
                'http://localhost:37497/api/recording',
                struct(
                        'parent_directory','/Users/neuroscientist/Documents/Data',
                        'base_text','experiment_01',
                        'append_text','mouse_a',
                        'default_record_engine','BINARY',
                        'start_new_directory','true'),
                weboptions('RequestMethod','put','MediaType','application/json'));

Use :code:`PUT /api/recording/<record_node_id>` to update a specific Record Node. Supported fields are :code:`parent_directory` and :code:`record_engine`.

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/recording/102",
                json={
                        "parent_directory": "/Users/neuroscientist/Documents/Data",
                        "record_engine": "BINARY",
                },
        )


Signal-chain configuration files
--------------------------------

Use :code:`GET /api/config` to fetch the current GUI configuration. The response is JSON with the XML payload stored in the :code:`info` field.

.. code-block:: python

        config = requests.get("http://localhost:37497/api/config").json()
        xml_text = config["info"]

Load a saved signal chain with :code:`PUT /api/load`:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/load",
                json={"path": "/Users/neuroscientist/Documents/OpenEphys/chain.xml"},
        )

Save the current signal chain with :code:`PUT /api/save`:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/save",
                json={"filepath": "/Users/neuroscientist/Documents/OpenEphys/chain.xml"},
        )

.. note:: :code:`/api/save` does not overwrite an existing file. It returns a message if the target path already exists.

Inspect processors, streams, and parameters
-------------------------------------------

Use :code:`GET /api/processors/list` to list the processor types that can be added to the graph:

.. code-block:: python

        available = requests.get("http://localhost:37497/api/processors/list").json()

Use :code:`GET /api/processors` to inspect the current signal chain:

.. code-block:: python

        graph = requests.get("http://localhost:37497/api/processors").json()

The response has the following structure:

.. code-block:: json

        {
                "processors": [
                        {
                                "id": 100,
                                "name": "File Reader",
                                "parameters": [],
                                "predecessor": null,
                                "streams": [
                                        {
                                                "name": "example_data",
                                                "source_id": 100,
                                                "sample_rate": 40000.0,
                                                "channel_count": 16,
                                                "parameters": []
                                        }
                                ]
                        }
                ]
        }

You can also query narrower endpoints:

* :code:`/api/processors/<processor_id>`
* :code:`/api/processors/<processor_id>/parameters`
* :code:`/api/processors/<processor_id>/parameters/<parameter_name>`
* :code:`/api/processors/<processor_id>/streams/<stream_index>`
* :code:`/api/processors/<processor_id>/streams/<stream_index>/parameters`
* :code:`/api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>`

.. note:: :code:`stream_index` is zero-based because the server indexes directly into each processor's stream list.

Parameter values are returned as strings in the JSON response, together with a :code:`type` field that describes the parameter kind.

Modify processors and parameters
--------------------------------

Processor-level parameters can be changed with :code:`PUT /api/processors/<processor_id>/parameters/<parameter_name>`.

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/processors/101/parameters/high_cut",
                json={"value": 6000},
        )

Stream parameters can be changed with :code:`PUT /api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>`.

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/processors/101/streams/0/parameters/enable_stream",
                json={"value": True},
        )

Accepted :code:`value` payloads are integers, floats, booleans, strings, and numeric arrays. Some parameters cannot be changed while acquisition is active; those requests return HTTP 400.

Use :code:`PUT /api/processors/<processor_id>/config` to send a processor-specific configuration message before starting acquisition:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/processors/100/config",
                json={"text": "NP REFERENCE 3 1 1 TIP"},
        )

To broadcast a message to all processors while acquisition is active, use :code:`PUT /api/message`:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/message",
                json={"text": "ACQBOARD TRIGGER 1 100"},
        )

.. tip:: Broadcast messages are saved by all Record Nodes, so they can be used to mark epochs within a recording.

The signal chain can also be edited remotely:

* :code:`GET /api/processors/clear` clears the graph.
* :code:`PUT /api/processors/delete` deletes a processor when given :code:`{"id": 101}`.
* :code:`PUT /api/processors/add` adds a processor when given :code:`{"name": "Bandpass Filter"}`.
* :code:`PUT /api/processors/add` also accepts :code:`source_id` or :code:`dest_id` to position the processor relative to an existing node.
* :code:`GET /api/undo` undoes the previous action.
* :code:`GET /api/redo` redoes the previous action.

Examples:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/processors/add",
                json={"name": "Bandpass Filter", "source_id": 100},
        )

        requests.put(
                "http://localhost:37497/api/processors/delete",
                json={"id": 101},
        )

Graph-editing endpoints that modify the signal chain are blocked while acquisition is active.

Performance endpoints
---------------------

Use :code:`GET /api/cpu` to retrieve the current audio callback CPU usage:

.. code-block:: python

        usage = requests.get("http://localhost:37497/api/cpu").json()

The returned JSON has the form:

.. code-block:: json

        {"usage": 0.12}

Use :code:`GET /api/latency` to inspect processor latency per stream:

.. code-block:: python

        latency = requests.get("http://localhost:37497/api/latency").json()

This returns one entry per processor, each with a list of stream names and their latency values.

Audio device control
--------------------

The audio endpoints let you inspect the available devices and change the currently selected device.

Use :code:`GET /api/audio/devices` to list available device types and names:

.. code-block:: python

        devices = requests.get("http://localhost:37497/api/audio/devices").json()

This returns JSON in the form:

.. code-block:: json

        {
                "devices": {
                        "ALSA": ["Device A", "Device B"],
                        "JACK": ["JACK Audio Connection Kit"]
                }
        }

Use :code:`GET /api/audio/device` to inspect the current device:

.. code-block:: json

        {
                "device_type": "ALSA",
                "device_name": "Device A",
                "sample_rate": 30000,
                "buffer_size": 512,
                "available_sample_rates": [30000, 44100, 48000],
                "available_buffer_sizes": [128, 256, 512, 1024]
        }

Use :code:`PUT /api/audio` to change any combination of :code:`device_type`, :code:`device_name`, :code:`sample_rate`, and :code:`buffer_size`:

.. code-block:: python

        requests.put(
                "http://localhost:37497/api/audio",
                json={
                        "device_type": "ALSA",
                        "device_name": "Device A",
                        "sample_rate": 30000,
                        "buffer_size": 512,
                },
        )

Close the GUI remotely
----------------------

To shut down the GUI, send an HTTP :code:`PUT` request to :code:`/api/quit`:

.. code-block:: python

        requests.put("http://localhost:37497/api/quit")

.. code-block:: matlab

        out = webwrite(
                'http://localhost:37497/api/quit',
                struct(),
                weboptions('RequestMethod','put','MediaType','application/json'));
