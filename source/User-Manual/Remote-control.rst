.. _remotecontrol:

Remote Control
##############

The **Open Ephys HTTP Server** enables remote control of the GUI via a RESTful API. The GUI runs an HTTP server on port 37497 ("EPHYS" on a phone keypad). 

Get/Set the GUI status
----------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/status`"

.. code-block:: js

    { 
        'mode' : 'ACQUIRE'
    }

Send a text message to the GUI
------------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/message`"

.. code-block:: js

    { 
        'text' : 'A text message sent to the GUI during acquisition' 
    }

Get processor information
-------------------------

.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/processors`"

.. code-block:: js

    {
        'processors' : [
            {
                'id' : 101,
                'name' : 'BandpassFilter',
                'predecessor': 100,
                'parameters' : [
                    {
                        'name' : 'low_cut',
                        'type' : 'Float',
                        'value' : 300.0
                    }, ...
                ],
                'streams' : [
                    {
                        'name' : 'ProbeA.AP',
                        'source_id' : 100,
                        'sample_rate' : 30000.0,
                        'channel_count' : 384,
                        'parameters' : [
                            {
                                'name' : 'FILTER',
                                'type' : BOOL,
                                'value' : false
                            }, ...
                        ]
                    }, ...
                ],

            }, ...
        ]
    }

Get/Set recording configuration
-------------------------------
        
.. csv-table:: 
   :widths: 10, 80

   "URL", ":code:`/api/recording`"

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

Set individual processor and stream parameters
----------------------------------------------

**GET** /api/processors/<processor_id>

**GET** /api/processors/<processor_id>/parameters

**GET** /api/processors/<processor_id>/parameters/<parameter_name>

**PUT** /api/processors/<processor_id>/parameters/<parameter_name>

**GET** /api/processors/<processor_id>/streams/<stream_index>

**GET** /api/processors/<processor_id>/streams/<stream_index>/parameters

**GET** /api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>

**PUT** /api/processors/<processor_id>/streams/<stream_index>/parameters/<parameter_name>

**PUT** /api/processors/<processor_id>/config

**PUT** /api/window

