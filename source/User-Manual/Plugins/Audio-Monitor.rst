.. _audiomonitor:
.. role:: raw-html-m2r(raw)
   :format: html

#################
Audio Monitor
#################

.. image:: ../../_static/images/plugins/audiomonitor/audiomonitor-01.png
  :alt: Annotated Audio Monitor settings interface

|

.. csv-table:: Streams data from incoming continuous channels to the computer's audio output.
   :widths: 18, 80

   "*Plugin Type*", "Utility"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Anjal Doshi, Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/development-juce6/Plugins/AudioMonitor"



Configuring your Arduino
=========================

This plugin works with any Arduino running the `Firmata <https://www.arduino.cc/en/reference/firmata>`__ firmware. To install this on your device, first make sure you've downloaded the `Arduino IDE <https://www.arduino.cc/en/main/software>`__ (version 1.0 or higher) for your platform of choice.

Next, connect the Arduino to your computer and upload the sketch in `File > Examples > Firmata > StandardFirmata`. This will allow your computer to control the state of the Arduino from software.

Plugin configuration
====================

You'll need a signal chain capable of generating TTL events. These can either come from a source node (such as the :ref:`rhythmfpga`, with the I/O board connected to the Digital Input port) or from a processor (such as the :ref:`phasedetector`). The Arduino Output is a sink, so it needs to be placed at the end of the signal chain. If you want to visualize your data with an :ref:`lfpviewer` as well, you'll need to use a :ref:`splitter` to split the data stream into two.

Before you start acquisition, select the device you want to use via the device selector drop-down menu. It will take a few seconds for the GUI to initialize communication with the board. The status of the connection will appear in the Message Center.

The other three drop-down menus determine the input ("Trig") channel that will translate software events into hardware outputs, the gate ("Gate") channel that determines whether or not the Arduino Output is active, and the output ("Output CH") channel of the Arduino that will be activated. 

All three of the these parameters can be changed while acquisition is active.


Default settings
----------------

* The Arduino can be triggered by any TTL event (regardless of the channel)

* There is no gate (the output will always be able to be activated)

* The output channel is 13 (the one tied to the LED on the Arduino Uno, useful for debugging purposes)

|
|


