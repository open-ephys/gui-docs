.. _acquisitionboard:
.. role:: raw-html-m2r(raw)
   :format: html

#################
Acquisition Board
#################

.. image:: ../../_static/images/plugins/acquisitionboard/acquisitionboard-01.png
  :alt: Annotated settings interface for the Acquisition Board plugin

.. csv-table:: Streams data from any generation of the `Open Ephys acquisition board <https://open-ephys.org/acq-board>`__.
   :widths: 18, 80

   "*Plugin Type*", "Source"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No; install via *Acquisition Board*"
   "*Key Developers*", "Josh Siegle, Aarón Cuevas López, Brandon Parks"
   "*Source Code*", "https://github.com/open-ephys-plugins/acquisition-board"


.. tip:: For more in-depth documentation on the Open Ephys Acquisition Board, please refer to the `Acquisition Board docs site <https://open-ephys.github.io/acq-board-docs/>`__.

Installing and upgrading
==========================

The Acquisition Board plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to open the Plugin Installer, browse to the "Acquisition Board", and click the "Install" button. After installation, *Acquisition Board* will appear in the processor list on the left side of the GUI's main window.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed. The plugin must be removed from the signal chain prior to upgrading.


Updating gateware
==========================

.. important:: Acquisition Boards with Open Ephys FPGAs may require a gateware update for this plugin to function properly.

The Acquisition Board plugin (version ≥1.1.3) will automatically check for a compatible gateware version on any connected devices. If the gateware needs to be updated, it will display a popup window with a link to the `Acquisition Board documentation site <https://open-ephys.github.io/acq-board-docs/User-Manual/Gateware-Update.html>`__, which contains updating instructions.

This only applies to Acquisition Boards with an Open Ephys FPGA (most boards purchased after December 2022). If you're unsure about which board you have, check out `this page <https://open-ephys.github.io/acq-board-docs/User-Manual/Generations-differences.html>`__.


Plugin configuration
====================

Headstages
############

On the left-hand side of the module, there are four rows with 2 slots each. Each row represents a headstage port on the acquisition board (denoted as A, B, C, & D left-to-right on the hardware). Each headstage port can accommodate up to two headstages*. When this processor is added to the signal chain, it automatically detects connected headstages, as well as whether they contain a 64- or 32-channel Intan chip, and whether or not they have an IMU (inertial measurement unit). Each slot in the row corresponding to the headstage port in use will be populated as follows:

- When a single headstage *without* an IMU is connected a port, the channel count of the Intan chip (32 or 64) will be displayed in a slot.
- When a single 3D capable headstage *with* an IMU is connected to a port, the channel count of the Intan chip (32 or 64) will be displayed in the first slot, and "IMU" will be displayed on the second slot. 
- When two headstages share a port, the channel count of each Intan chip (32 or 64) will be displayed in each slot. In this case, even if either or both headstages are 3D capable, no "IMU" will be displayed, and 3D capabilities won't be available.

..  note:: Plugging two headstages into one port requires a `dual headstage adapter <https://open-ephys.github.io/acq-board-docs/Hardware-Guide/Cables.html#dual-headstage-adapter>`__. The SPI connector labelled 1 corresponds to the headstage that appears in the first slot of the row, and the the connector labelled 2, to the second slot.

If you add or remove headstages after the Acquisition Board processor has been loaded, you must press the "RESCAN" button for the hardware changes to be detected.

*Actually, each headstage port can accommodate up to two 64-channel Intan chips that can be on the same headstage, but having more than one chip per headstage is rare, so this is described per headstage.*

Using 16-channel headstages
----------------------------

Clicking on the slot of one of the detected headstages will toggle it between 32-channel and 16-channel mode. This is necessary because the difference between 16-channel and 32-channel headstages cannot detected in software, and has to be selected manually.

Sample rate selection
#######################

Sample rates between 1 and 30 kHz can be selected with this drop-down menu. This will determine the sample rate for all headstages, digital inputs, and ADCs.

Bandwidth interface
#####################

Used to determine the settings for the analog high and low cut filters on the Intan chip. Because only certain values are possible for each, the number that appears may be different from the number you typed in; the chip will automatically select the nearest value, and that will be indicated in the GUI. 

Turning on AUX channels
#######################

Pressing the "AUX" button toggles the headstage AUX inputs on and off. Each Intan chip can communicate with up to 3 "auxiliary" inputs, which are connected to a 3-axis accelerometer on some headstages. If the button is off (gray), the AUX channels on each headstage will be ignored. If the button is on (yellow), they will be sent as parallel data channels, with the same sampling rate as the neural data.

Turning on ADC channels
########################

Pressing the "ADC" button toggles the ADC inputs on and off. If the button is off (gray), the ADC channels acquired by the board will be discarded. If the button is on (yellow), they will be sent as parallel data channels, with the same sampling rate as the neural data. To use these channels, an I/O board needs to be connected to the second HDMI port from the left.

Audio output
#############

The "AUDIO OUT" interface will allow you select up to 2 channels to preview in real time, directly from the acquisition board. There is an audio jack on the board that's mirrored to the first two analog output channels (CH1 and CH2 of the Digital & Analog I/O for L and R sound outputs when connected to HDMI port for analog output — it is the leftmost HDMI port). To select a channel for monitoring, click on one of the two "Audio Out" buttons to reveal the pop-up channel selector.

.. note:: In general, we recommend using the :ref:`audiomonitor` plugin to listen to spikes through your computer's speakers, as it is much more flexible.

Noise slicer
##############

Sets the threshold for the noise slicer on the hardware audio outputs (sets any values below threshold to zero, to improve the signal-to-noise ratio). In practice, this doesn't work particularly well.

Clock divider
##############

The BNC connector on the back of the board will send a digital pulse each time a new sample is acquired. The clock divider makes it possible to downsample this clock, so a pulse is sent every *N* samples. Note that *N* can only be one or an even number, even though this is not enforced via the settings interface.

DSP button
###########

in addition to the analog filters, the Intan chips also have an on-chip DSP high-pass filter for removing the DC offset on each channel. This can be toggled on and off by clicking the "DSP" button (yellow = on), and the cutoff frequency can be changed by typing a value into the associated text box. Again, there are only certain values for the cutoff frequency; the chip will choose the one closest to what you selected and return this value to the GUI.

DAC TTLs
##########

When this button is on, the digital-to-analog converter (DAC) will generate TTL outputs whenever the output signal crosses a threshold. Note that this is an untested feature and not recommended for experiments.

DAC High-pass filter
######################

Sets the high-pass filter cutoff frequency for the DAC outputs.

TTL settle
###########

Ties one of the digital inputs on the acquisition board/evaluation board to the "fast-settle" functionality of the Intan chips. If the selected digital input channel goes high, it will trigger the reset of the amplifiers across all headstages.

LED button
###########

The button labeled "LED" will turn the board LEDs on or off.

Impedance testing
##################

To open the impedance measurement interface, click the "window" or "tab" buttons at the top of the plugin editor. This will bring up an impedance measurement interface that looks like this:

.. image:: ../../_static/images/plugins/acquisitionboard/acquisitionboard-02.png
  :alt: Annotated impedance measurement interface

Memory Monitor
###############

..  note:: This pertains only to Acquisition Board Gen 2 and Gen 3 with firmware 1.5.1+

.. image:: ../../_static/images/plugins/acquisitionboard/acquisitionboard-04.png
  :alt: Updated editor with a memory usage monitor on the left

While data is waiting to be transferred from the acquisition board to the computer, it sits in the
acquisition board's buffer. The memory monitor on the left provides a visualization of how much data
has accumulated in this buffer as a percentage of its total capacity. The memory monitor should stay
at or near zero. Accumulated data in the hardware's buffer indicates an error that is causing data
to be read from the hardware too slowly. This might eventually halt the program if the buffer fills
to its capacity.

|

Closed-loop feedback
====================


Installing the "Acquisition Board" will also install the "Acq Board Output" plugin, which can be used to trigger the digital outputs of the acquisition board.

.. image:: ../../_static/images/plugins/acquisitionboard/acquisitionboard-03.png
  :alt: Annotated Acq Board Output plugin

If this plugin is placed downstream of the Acquisition Board plugin, as well as a plugin that generates TTL events (e.g., :ref:`crossingdetector` or :ref:`rippledetector`), the digital output channel specified by the :code:`TTL_OUT` parameter will be temporarily set to high each time a TTL event is received on the :code:`TRIGGER_LINE`. The approximate duration of this event (in milliseconds) is set by the :code:`EVENT_DURATION` parameter. 

This configuration can be used to perform closed-loop feedback experiments in which some feature of the neural data (such as phase of an oscillation, or the presence of a ripple event), is used to trigger stimulation.