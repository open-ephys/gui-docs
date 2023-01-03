.. _ttlpanels:
.. role:: raw-html-m2r(raw)
   :format: html

################
TTL Panels
################

.. image:: ../../_static/images/plugins/ttlpanels/ttlpanels-01.png
  :alt: Annotated TTL Panels Editor

.. csv-table:: .
   :widths: 18, 80

   "*Plugin Type*", "Filter, Sink"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Christopher Thomas"
   "*Source Code*", "https://github.com/open-ephys-plugins/spectrum-viewer"

.. note:: Provides toggle and display panels for debugging TTL events.

Installing and upgrading
###########################

The TTL Panels plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** to access the Plugin Installer, browse to the "TTL Panels" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.

Plugin Configuration
######################

First, make sure the correct sub-processor is selected in the plugin editor. Then, select one or two channels to display. The first channel (A) will appear in yellow. The second channel (B) will appear in green.

You can update the selected channels, but not the selected sub-processor, while acquisition is active.

Spectrum Display
#################

.. image:: ../../_static/images/plugins/spectrumviewer/spectrumviewer-02.png
  :alt: Annotated Spectrum Viewer Editor

The Spectrum Viewer analyzes the incoming data in 250 ms segments, and overlays the resulting power spectra for 5 adjacent segments. The power is plotted on a log scale, and the frequency range is fixed between 4 and 500 Hz.

We are planning to add more features to the spectrum display over time.

|