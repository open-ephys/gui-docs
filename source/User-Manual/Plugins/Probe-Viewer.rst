.. _probeviewer:
.. role:: raw-html-m2r(raw)
   :format: html

################
Probe Viewer
################

.. image:: ../../_static/images/plugins/probeviewer/probeviewer-01.png
  :alt: Annotated Probe Viewer editor

.. csv-table:: Visualizes the signal from a linear probe as a heatmap (time x channels). Can be configured to show RMS signal, spike rate, or power in different frequency bands.
   :widths: 18, 80

   "*Plugin Type*", "Sink"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Josh Siegle, K. Michael Fox, Anjal Doshi"
   "*Source Code*", "https://github.com/open-ephys-plugins/probe-viewer"


Installing and upgrading
###########################

The Probe Viewer plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to access the Plugin Installer, browse to the "Phase Calculator" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.

Recommended signal chain
#########################

The Probe Viewer works best with data acquired by linear silicon probes (especially Neuropixels), but it can be used with any type of input source. It can only visualize data from one stream at a time, but the stream can be changed while acquisition is active (this feature is new in v0.6.x).

Plugin configuration
#####################

The Probe Viewer visualizer interface (accessed by clicking the "tab" or "window" buttons at the top right of the plugin editor) looks like this:

.. image:: ../../_static/images/plugins/probeviewer/probeviewer-02.png
  :alt: Annotated Probe Viewer visualizer

To change the subset of channels to be viewed, drag the edges of the bounding box in the channel browser interface on the left-hand side of the visualizer. The currently displayed channels are shown as circular icons in the zoomed-in view to the right.

Render modes
------------

1. **RMS Signal** - Color corresponds to the overall amplitude of the signal on each channel (computed as root-mean-squared signal amplitude within each pixel window). The minimum and maximum of the color scale can be adjusted.

2. **Freq. Band Power** - Color corresponds to the spectral power at a given center frequency. For example, this can be used to visualize theta-band (8 Hz) power across depth.

3. **Spike Rate** - Color corresponds to the number of threshold crossings per unit time within each pixel window. The threshold voltage can be adjusted.

|
|

