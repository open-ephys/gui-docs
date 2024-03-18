.. _ephyssocket:
.. role:: raw-html-m2r(raw)
   :format: html

#####################
Ephys Socket
#####################

.. image:: ../../_static/images/plugins/ephyssocket/ephyssocket-01.png
  :alt: Annotated Ephys Socket Editor

.. csv-table:: Receives continuous data from the OpenCVMatTCPClient in Bonsai. This is intended to be a quick way to stream ephys data from Bonsai, for example if you're performing closed-loop feedback in Bonsai, but want to visualize your data in the Open Ephys GUI.
   :widths: 18, 80

   "*Plugin Type*", "Source"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Jon Newman"
   "*Source Code*", "https://github.com/open-ephys-plugins/ephys-socket"

Installing and upgrading
###########################

The Ephys Socket plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to access the Plugin Installer, browse to the "Ephys Socket" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.

Plugin Configuration
######################

In Bonsai
-----------

#. Make sure you have the "**Bonsai.Ephys**" and "**OpenEphys.Sockets.Bonsai**" packages installed. For the latter, you'll have to set the package source to "Community Packages".

#. Create a signal chain that includes (at least):

   * A source that generates an OpenCV.Net.Mat output (e.g., Rhd2000EvalBoard, which is compatible with the Open Ephys acquisition board)

   * A member selector that isolates the OpenCV.Net.Mat output

   * The CreateTcpServer and SendTcpData nodes from OpenEphys.Sockets.Bonsai.

#. Click on the CreateTcpServer, and edit the fields there:

   * Set the address (for communication within the same computer, put "localhost" without quotes)

   * Add a name to use for the connection channel. This can be an arbitrary name, but must match the same name used in the SendTcpData node

   * Change the port number to :code:`9001`

#. (Optional) Add a "Select Channels" operator before SendTcpData, to select a subset of channels for streaming.

#. Click on the SendTcpData node, and add the name of the connection initially set in the CreateTcpServer node. Note that the name displayed in Bonsai for the CreateTcpServer node will have changed to the name of the communication channel.

#. Start the workflow to initiate the data stream.

The workflow should look something like the one pictured below (:code:`.bonsai` file available in the `Ephys Socket GitHub repo <https://github.com/open-ephys-plugins/ephys-socket/tree/main/Resources>`__):

.. image:: ../../_static/images/plugins/ephyssocket/ephyssocket-02.png
  :alt: A compatible Bonsai workflow using hardware


A second workflow is included in the same `Resources folder <https://github.com/open-ephys-plugins/ephys-socket/tree/main/Resources>`__ which uses a function generator (imported from Bonsai.Dsp). This workflow is useful for testing socket connections without requiring hardware to be connected:

.. image:: ../../_static/images/plugins/ephyssocket/ephyssocket-03.png
  :alt: A compatible Bonsai workflow using a function generator


In Open Ephys
--------------

#. Drag and drop the Ephys Socket plugin onto the signal chain.

#. Add downstream plugins (e.g., LFP Viewer).

#. Click on the port number, and make sure it matches the same number used in Bonsai.

#. Click the "Connect" button to establish the connection with Bonsai. If it's not connecting, make sure the Bonsai workflow is running, and the port numbers match.

#. Ensure the "frequency" parameter matches the sample rate of the Bonsai data stream. By default the Open Ephys acquisition board acquires data at 30000 Hz.

#. You shouldn't have to change the "offset", or "scale" parameters unless you are interfacing with a custom data stream.

#. Start acquisition in Open Ephys. If all is working, you should see the data stream in the LFP Viewer.

.. warning:: This plugin doesn't currently guarantee that samples won't be lost in transit, so it's essential to save your data stream on the Bonsai end. We are working on implementing a buffering system on the Open Ephys side, to ensure that all samples are being received. If you suspect samples are being dropped (e.g., if the LFP Viewer is not updating at the expected speed), sending fewer channels at a time should fix the problem.


Header Format for Custom Data Streams
######################################

If something other than the SendTcpData node (found in OpenEphys.Sockets.Bonsai, see :ref:`In Bonsai` for details) is used to stream data (e.g., a Python script that sends data over the TCP socket), a header must be prepended to the data stream for Ephys Socket to interpret the data.

An example Python script is included in the `Resources <https://github.com/open-ephys-plugins/ephys-socket/tree/main/Resources>`__ folder of the plugin repository, which implements the format described here. Each variable is 4 bytes long (with the exception of the Bit Depth, which is 2 bytes long), and must be sent in the order listed below. The total header length is 22 bytes.

.. csv-table:: Ephys Socket header variables
    :widths: 18, 10, 60
    
    "**Variable**", "**Data Type**", "**Description**"
    "Offset", "`int32`", "Integer defining the offset in the data stream. For TCP sockets, value must always be set to **0**."
    "Number of Bytes", "`int32`", "Total number of bytes sent in each packet. Calculated as :code:`num_bytes = num_channels * num_samples * element_size`. This does **not** include the header bytes."
    "Bit Depth", "int16", "Depth defines the OpenCV.Mat `Depth <https://github.com/horizongir/opencv.net/blob/main/src/OpenCV.Net/CoreTypes.cs#L93>`__, an enumeration defined as :code:`[U8, S8, U16, S16, S32, F32, F64]`, where U is 'unsigned', S is 'signed', F is 'float', and the number indicates the number of bits. For example, :code:`U16` means 'unsigned integer with 16 bits'. Note that the enumeration is zero-indexed, where :code:`U8 = 0`, and :code:`F64 = 6`"
    "Element Size", "`int32`", "Number of bytes needed for each sample. For :code:`U16`, :code:`element_size = 2`, while for :code:`F64`, :code:`element_size = 8`."
    "Number of Channels", "`int32`", "Number of channels per packet."
    "Number of Samples", "`int32`", "Number of samples sent per channel per packet."

