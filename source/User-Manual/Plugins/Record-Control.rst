.. _recordcontrol:
.. role:: raw-html-m2r(raw)
   :format: html

################
Record Control
################

.. image:: ../../_static/images/plugins/recordcontrol/recordcontrol-01.png
  :alt: Annotated Record Control editor

.. csv-table:: Allows recording to be automatically toggled on and off by TTL inputs, instead of manually pressing the record button. It can be used to trigger recording using external TTLs, or based on events that are detected in a continuous data stream.
   :widths: 18, 80

   "*Plugin Type*", "Utility"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Josh Siegle, Aarón Cuevas López"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/main/Plugins/RecordControl"

Plugin configuration
#####################

First, make sure the correct TTL trigger line is selected (the line labels use 1-based indexing). Then check the desired edge type ("Rising" or "Falling").

Acquisition must be running for the Record Control plugin to respond to TTL events. Record Control modules can be dropped anywhere in the signal chain as long as there's a data source upstream, and data will pass through them unchanged.

.. caution:: If trigger events are too close together, the Record Nodes may not have enough time to open and close the relevant files. There should be at least one second between events that toggle the recording state.

Trigger modes 
-------------

* **Edge set**: Starts recording when the selected edge (rising or falling) is detected, and stops recording when the opposite edge is detected. Use this mode to record data whenever a particular TTL line is "high" or "low."

* **Edge toggle**: Starts recording when the selected edge (rising or falling) is detected, and stops recording when the next edge of the same type is detected. Use this mode to toggle recording state using incoming events on a particular TTL line.

|


