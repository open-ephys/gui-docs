.. _eventbroadcaster:
.. role:: raw-html-m2r(raw)
   :format: html

#################
Event Broadcaster
#################

.. image:: ../../_static/images/plugins/eventbroadcaster/eventbroadcaster-01.png
  :alt: Annotated Event Broadcaster editor

|

.. csv-table:: Sends events from the GUI to external applications.
   :widths: 18, 80

   "*Plugin Type*", "Sink"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Christopher Stawarz"
   "*Source Code*", "https://github.com/open-ephys-plugins/EventBroadcaster"

Plugin configuration
######################

**Restart connection**: Resets the ZeroMQ data streaming backend

**Port number**: Modifies the port number from which this plugin will send data 

**Format**: Select the event packet format (:code:`Header/JSON` recommended)

Packet format
##############

Spikes
-------

.. code-block:: 

   {
      "type" : "spike",
      "sortedID" : 0,
      "numChannels" : 2,
      "threshold" : [50, 50],
      "timing" : {
         "sampleRate" : 40000,
         "timestamp" : 372649
      },
      "identifier" : "spikesource",
      "name" : "ST  p102.0 n3",
      "metadata" : {}
   }

TTL Events
----------

.. code-block:: 

   {
      "type" : "ttl",
      "channel" : 0,
      "data" : True,
      "timing" : {
         "sampleRate" : 40000,
         "timestamp" : 594829
      },
      "identifier" : "dataderived.phase.peak.positive",
      "name" : "Phase detector output 1",
      "metadata" : {}
   }

Receiving data in Python
########################

See the `open-ephys-python-tools <https://github.com/open-ephys/open-ephys-python-tools/blob/main/open_ephys/streaming/>`__ repository for a more complete implementation.

.. code-block:: python

   import zmq
   import json

   url = "tcp://localhost:5557"
        
   context = zmq.Context()
   socket = context.socket(zmq.SUB)
   socket.connect(url)

   for eventType in (b'ttl', b'spike'):
      socket.setsockopt(zmq.SUBSCRIBE, eventType)

   def callback(info):
      #do something with the incoming data
      print(info)

   while True:
      parts = socket.recv_multipart()

      event_info = json.loads(parts[1].decode('utf-8'))

      callback(event_info)


