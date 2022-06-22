.. _zmqinterface:
.. role:: raw-html-m2r(raw)
   :format: html

################
ZMQ Interface
################

.. image:: ../../_static/images/plugins/zmqinterface/zmqinterface-01.png
  :alt: ZMQ interface

|

.. csv-table:: Sends continuous data, events and spikes to external applications using the ZeroMQ library, making it possible to create advanced visualization and monitoring add-ons, such as `OPETH <https://github.com/hangyabalazs/opeth>`__ 
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Francesco Battaglia, András Széll"
   "*Source Code*", "https://github.com/open-ephys-plugins/zmq-interface"

.. tip:: The client application may be written in any language/platform supporting ZeroMQ.

Plugin Configuration
#####################

The plugin consists for 3 main parameters in the editor

#. **Stream:** The stream's data to send.

#. **Channels:** List of continuous data channels to send (all selected by default)

#. **Data Port:** Port number to send data to.

The editor also shows a list of clients connected to the plugin via the listen port (5557). If a client is successfully connected, then it's name will show up in green in the list once acquisition has started. If the client disconnects in the middle of acquiring data, the name will turn red.

Data Packets
################

The data packets sent by ZMQInterface are multipart messages divided into three parts

Message Envelope:
-----------------
The type of message being received, either DATA or EVENT.

Message Header:
-----------------
An array of message header attributes and its values:

  .. csv-table::
   :widths: 15, 100

   "**Continuous**", "message number, type (data), content = {stream id, channel number, number of samples, number of real samples, timestamp (sample number), sample rate}, data size"
   "**TTL Event**", "message number, type (TTL), content = {stream id, sample number, event stream, event channel}, data size"
   "**Spike**", "message number, type (spike), spike = {type(single, steretrode, tetrode), sample number, number of channels, num of samples, pre preak samples, post peak samples, electrode id, spike stream}"


Message Data:
-------------
The actual continuous data.

  .. csv-table::
   :widths: 15, 50

   "**Continuous**", "Continuous data from a channel"
   "**TTL Event**", "Event data (in order) = {1Byte\: 'Event Line', 1 Byte\: 'Event state(0 or 1)', 8 Bytes:'TTL Word'}"
   "**Spike**", "Spike data"

Example Code
#############

The example code for receiving continuous data is written in Python, although in principle it should be possible from any language supporting ZeroMQ. 
Currently, ZMQ Interface doesn’t support receving event data. If you want to receive event data in Open Ephys using ZMQ, use the :ref:`networkevents` plugin.

In the python_clients/ZMQPlugins directory of the original repo, you'll find an example python client called simple_plotter_zmq.py which plots the continuous data received from the Open Ephys GUI over a network port (see image below).


.. image:: ../../_static/images/plugins/zmqinterface/zmqinterface-02.png
  :alt: ZMQ Client in Python

There are two important things that needs to be taken care of by the client app. One is, setup a heartbeat protocol to maintain connection between Open Ephys GUI and the client app. The other is receiving the data from the GUI and handling it in the client app. 






