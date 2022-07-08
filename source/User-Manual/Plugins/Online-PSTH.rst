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

The Online SPTH plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to open the Plugin Installer, browse to the "Online PSTH" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.


Plugin configuration
======================

The Online PSTH plugin won't work unless there is at least one event-generating plugin and one spike-generating plugin upstream from it in the signal chain. Assuming this is true, use the following steps to configure the plugin:

1. Select the TTL line to use for triggering.

2. Enter the histogram bin size (default: 25 ms).

3. Enter the pre-event and post-event window size (default: 500 ms).


Plugin usage
=============

* Use the buttons in the upper-right of the editor to open the visualizer in a separate tab or window to scroll through detected electrodes.

* Mouse over data points to view the corresponding time window and number of spikes in a given bin.

* Press the "clear" button in the top left of the window to reset the histograms.

|



