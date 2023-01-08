.. _onlinepsth:
.. role:: raw-html-m2r(raw)
   :format: html


#########################
Online PSTH
#########################

.. image:: ../../_static/images/plugins/onlinepsth/onlinepsth-01.png
  :alt: Annotated Event Triggered Average settings interface


.. csv-table:: Aligns spike times to incoming TTL events in order to create a peri-stimulus time histogram display.
   :widths: 18, 80

   "*Plugin Type*", "Sink"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Clayton Barnes, Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys-plugins/online-psth"


Installing and upgrading
==========================

The Online PSTH plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to open the Plugin Installer, browse to the "Online PSTH" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.


Plugin configuration
======================

The Online PSTH plugin won't work unless there is at least one event-generating plugin and one spike-generating plugin upstream from it in the signal chain. Assuming this is true, use the following steps to configure the plugin:

Define the trigger conditions
------------------------------

Click the "Configure" button to bring up the trigger condition interface:

.. image:: ../../_static/images/plugins/onlinepsth/onlinepsth-02.png
  :alt: Trigger condition interface

By default, there are no conditions defined. Click the "+" button to create a new condition. Each condition has the following parameters:

- *Name*: can be edited by double-clicking
- *TTL Line*: select from lines 1-16 by clicking once on the line name
- *Type*: click on the icon to change the trigger type

The Online PSTH plugin will automatically create a separate histogram for each condition + electrode combination:

.. image:: ../../_static/images/plugins/onlinepsth/onlinepsth-03.png
  :alt: Example histograms

Triggering methods
###################

The "type" of each condition defines how individual trials are triggered:

- :code:`TTL`: (default) every "ON" TTL event on the specified line triggers a new trial
- :code:`TTL + MSG`: a new trial is only triggered by the specified TTL line after receiving a broadcast message containing the condition name. Broadcast messages can be sent using the HTTP server, or by entering a string into the message center at the bottom of the GUI. This option makes it possible to define many unique conditions using only one TTL input line.
- :code:`MSG`: a new trial is triggered by broadcast messages containing the condition name. The timing of the trial start will be accurate to within one software buffer length (typically around 20 ms)


Define the window size
------------------------------

It's possible to change the the histogram bin size (default: 25 ms) and pre-event and post-event window size (default: 500 ms) at any time. Note that if the histogram display has been active for many trials without clearing the spikes, recalculating the histograms may be slow.


Select the display type
--------------------------

The visualizer can display data as histograms, spike rasters, or lines, with the option to overlay the rasters on top of the histograms or lines.


Plugin usage
=============

* Use the buttons in the upper-right of the editor to open the visualizer in a separate tab or window to scroll through detected electrodes.

* Mouse over data points to view the corresponding time window and average firing rate in a given bin.

* Press the "clear" button in the top left of the window to reset the histograms.

|



