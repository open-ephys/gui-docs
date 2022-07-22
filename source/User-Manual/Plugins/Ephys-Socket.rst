.. _ephyssocket:
.. role:: raw-html-m2r(raw)
   :format: html

#####################
Ephys Socket
#####################

.. image:: ../../_static/images/plugins/ephyssocket/ephyssocket-01.png
  :alt: Annotated Ephys Socket Editor

.. csv-table:: Receives continuous data from the OpenCVMatSocket in Bonsai. This is intended to be a quick way to stream ephys data from Bonsai, for example if you're performing closed-loop feedback in Bonsai, but want to visualize your data in the Open Ephys GUI.
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

#. Make sure you have the "**Bonsai.Ephys**" and "**Bonsai.JonsUtils**" packages installed. For the latter, you'll have to set the package source to "Community Packages".

#. Create a signal chain that includes (at least):

   * A source that generates an OpenCV.Net.Mat output (e.g. Rhd2000EvalBoard, which is compatible with the Open Ephys acquisition board)

   * A member selector that isolates the OpenCV.Net.Mat output

   * The OpenCVMatSocket from JonsUtils.

#. Click on the OpenCVMatSocket, and change the port number to :code:`9001`.

#. (Optional) Add a "Select Channels" operator before OpenCVMatSocket, to select a subset of channels for streaming.

#. Start the workflow to initiate the data stream.

The workflow should look something like the one pictured below (:code:`.bonsai` file available in the `Ephys Socket GitHub repo <https://github.com/open-ephys-plugins/ephys-socket/tree/main/Resources>`__):

.. image:: ../../_static/images/plugins/ephyssocket/ephyssocket-02.png
  :alt: A compatible Bonsai workflow


In Open Ephys
--------------

#. Drag and drop the Ephys Socket plugin onto the signal chain.

#. Add downstream plugins (e.g., LFP Viewer)

#. If the downstream plugins are grayed out, click the "Connect" button to re-establish the connection with Bonsai. If it's not connecting, make sure the Bonsai workflow is running, and the port numbers match.

#. Make sure the "channels" parameter matches the number of channels coming out of Bonsai. You shouldn't have to change the "samples", "offset", or "scale" parameters unless you are interfacing with a custom data stream.

#. Ensure the "frequency" parameter matches the sample rate of the Bonsai data stream. By default the Open Ephys acquisition board acquires data at 30000 Hz.

#. Start acquisition in Open Ephys. If all is working, you should see the data stream in the LFP Viewer. If the buffer size (number of channels) doesn't match, Open Ephys won't be able to acquire data.

.. warning:: This plugin doesn't currently guarantee that samples won't be lost in transit, so it's essential to save your data stream on the Bonsai end. We are working on implementing a buffering system on the Open Ephys side, to ensure that all samples are being received. If you suspect samples are being dropped (e.g., if the LFP Viewer is not updating at the expected speed), sending fewer channels at a time should fix the problem.