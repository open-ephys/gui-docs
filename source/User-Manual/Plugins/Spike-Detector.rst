.. _spikedetector:
.. role:: raw-html-m2r(raw)
   :format: html

################
Spike Detector
################

.. image:: ../../_static/images/plugins/spikedetector/spikedetector-01.png
  :alt: Annotated Spike Detector settings interface

|

.. csv-table:: Detects spikes in continuous data and packages them as events that can be read by downstream processors.
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/master/Plugins/BasicSpikeDisplay/SpikeDetector"

Plugin configuration
######################

To initialize the spike detector, you must configure the electrode layout. Here are the steps to follow:

#. Select the type of electrode from the drop-down menu (you can have multiple types of electrodes in the same configuration).

#. Use the arrow buttons to select the number of electrodes of that particular type.

#. Hit the plus button to add those electrodes to the configuration.

#. Select the name of the electrode from the lower drop-down menu.

#. Change the settings for that electrode by one of the following:

	#. Update the name by typing in the drop-down menu.

	#. Inactivate individual channels by deselecting them to the right of the drop-down menu.

	#. Change the channel mapping by clicking the "EDIT" button and opening the "channel drawer."

	#. Change the threshold by clicking the "EDIT" button and moving the slider on the right.

#. Individual electrodes can be removed by clicking the "DELETE" button.

#. The "MONITOR" button sends the channels of that electrode to the audio monitor.

Once the electrode layout has been configured, the Spike Viewer will automatically update the layout of its spike plots. Within the Spike Viewer, you can change a second threshold to filter incoming spikes by peak height. This threshold is completely independent of the one in the Spike Detector, so we recommend setting a low threshold in the Spike Detector, then configuring the Spike Viewer threshold by eye.




