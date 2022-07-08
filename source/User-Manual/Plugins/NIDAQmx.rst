.. _nidaqmx:
.. role:: raw-html-m2r(raw)
   :format: html

NIDAQmx
===========

.. image:: ../../_static/images/plugins/nidaqmx/nidaqmx-01.png
  :alt: NIDAQmx plugin settings interface

.. csv-table:: Streams analog and digital data from National Instruments (NI) hardware. Use multiple instances of this plugin to acquire data from several PXI-, PCI-, and/or USB-based NI devices simultaneously. Can be used in tandem with the Neuropix-PXI plugin, to read in analog and digital inputs in parallel with Neuropixels data.
   :widths: 18, 80

   "*Plugin Type*", "Source"
   "*Platforms*", "Windows only"
   "*Built in?*", "No"
   "*Key Developers*", "Pavel Kulik"
   "*Source Code*", "https://github.com/open-ephys-plugins/nidaq-plugin"


Requirements
#######################

* One National Instruments data acquisition (DAQ) device (PXI, PCI, or USB); see below for a list of devices we've tested.

* (optional) An NI terminal block (e.g. BNC-2090A or BNC-2110) and compatible cables

* Installed NI-DAQmx drivers (available for download `here <https://www.ni.com/en-us/support/downloads/drivers/download.ni-daqmx.html#348669>`__; version 19.0+ recommended).

* **For synchronization**: One digital input line shared between all data sources streaming data to the GUI (e.g., Open Ephys acquisition board, Neuropixels PXI basestation, other NI devices).


Compatible hardware
#####################

So far, we've tested the following devices with this plugin:

.. csv-table::
   :widths: 30, 30, 30

   "**Device**", "**Inputs**", "**Price**"
   "PXI-6133", "8 analog, 8 digital", "$3643"
   "PXIe-6341", "16 analog, 24 digital", "$1239"
   "PXI-6521", "16 digital (24 V logic)", "$715"
   "PCIe-6321", "16 analog, 24 digital", "$746"
   "USB-6001", "8 analog, 13 digital", "$220"

If you're using the NIDAQmx plugin alongside the :ref:`neuropixelspxi` module, we recommend using a PXI-compatible DAQ in the same chassis as the Neuropixels PXI basestation.

.. tip:: All of these devices (with the exception of **PXI-6133**) are known to experience "signal ghosting" if unused analog input channels are not grounded. Grounding at least one positive terminal of an unused analog input channel can reduce ghosting by ~99% of the highest amplitude active channel. 


Connecting to your NIDAQ hardware
##################################

When you drag and drop the NIDAQmx plugin from the Processor List onto the Editor Viewport, it automatically checks for all connected NI devices. If multiple devices are found, a 'Device Swap' button will appear in the top right corner of the plugin editor that acts as a toggle switch to cycle through all available devices. The module only looks for devices when the plugin initially is dropped into the EditorViewport. If a new device is connected after the plugin is dropped, you will need to drop a new NIDAQmx module to be able to connect to the new device.

The editor will automatically generate control buttons for each analog and digital input available on the selected NI device. Each analog channel contains a channel status toggle button as well as a terminal configuration toggle button. The channel status toggle button is set to on (green) by default for all channels and is intended to enable/disable channels on the fly. Toggling an enabled channel to off (gray) will 'zero-out' the data coming in on that channel. 

Each analog channel also contains a terminal configuration toggle specific to the device connected. Possible terminal device configurations are RSE (Referenced Single-Ended), NRSE (Non-Referenced Single-Ended), DIFF (Differential), and PDIF (Pseudo-Differential). The terminal configuration toggle setting in the module should match the configuration in your experimental setup. Since most NI devices cannot infer the terminal configuration electronically, it is up to the user to ensure the module configuration matches the experimental setup prior to acquiring and recording data. An inconsistent terminal configuration can affect the amplitude and voltage offset of the incoming data.

Each digital channel contains a channel status toggle button as well. Disabling digital channels is particularly useful when a digital input line is constantly high or a digital line is not used at all. 

On the right side of the module are pull-down selection menus to select the sample rate and voltage range of all analog input channels (set individually for each channel). The module defaults to the highest sample rate as well as the largest voltage range available on the devices. Sample rate and voltage range selection are both disabled during data acquisition and recording. 


Offline Synchronization
#########################################

The NIDAQmx module has its own sample clock, even if it's in the same PXI chassis as other hardware (e.g. a Neuropixels basestation). Therefore, it needs to be treated as a separate device when performing offline synchronization.

To align timestamps across devices, one device has to be treated as the "master" clock, and all other sets of timestamps must be scaled and shifted to match this clock. This requires that a digital sync line is physically connected across all devices. 

While it's possible to use one of the NI devices (or a Neuropixels PXI basestation) to generate the sync pulses, we recommend using an Arduino running the `sync-barcodes <https://github.com/open-ephys/sync-barcodes>`__ program, which generates unique temporal barcode patterns every 30 s. This will allow you to align sample clocks unambiguously, even if your devices are stopped and started at different times.

To perform the offline alignment, you'll need to identify the first and last shared sync pulse in the recording. The temporal offset between the start of the first sync pulses defines the shift between any device and the master clock. Once you know this, you can calculate the expected interval between the first and last sync pulse (given the expected sample rate of a device). The ratio of the actual interval to the expected interval defines the scaling factor for that device.

Then, each non-master clock can be transformed by the following equation (in Matlab):

.. code-block:: matlab

  aligned_timestamps = scaling .* original_timestamps + shift;

The "aligned_timestamps" will now be aligned to the master clock, and are ready for further analysis.


Online Synchronization
#########################################

An Open Ephys Record Node can automatically synchronize data sources that share the same physical sync signal. Since each device acquires data asynchronously, a shared clock signal between two different devices will have different timestamps for corresponding TTL events. A Record Node can designate one of these channels as the main ‘synchronization channel’ and scale the timestamps coming auxiliary devices to be in alignment with the main device. 

For Neuropixels probes, you can use the Neuropixels-PXI as the main synchronization device by physically connecting the SMA output of the Neuropixels basestation to any digital input channel on the NIDAQ device. 

In Open Ephys, place a :ref:`merger` before the Record Node and connect both the :ref:`neuropixelspxi` plugin and the NIDAQmx plugin. The Record Node will show either one or two subprocessors per Neuropixels probe (depending if the 2.5kHz LFP band is included by the probe model) and one subprocessor for the NIDAQ device. Each subprocessor’s sync channel monitor will turn green if the digital line on that subprocessor is synchronized with any of the other subprocessors coming into that Record Node. 

.. image:: ../../_static/images/plugins/nidaqmx/NIDAQ_NPXMerged.png
  :alt: NIDAQmx plugin syncing

Upon starting acquisition, the first and third sync channel monitors in the Record Node turn green first, as these contain the 30 kHz AP band of the probes and the source of the synchronization signal. Shortly after, the fifth sync channel monitor turns green, which contains the sync signal coming into the NIDAQ device from the Neuropixels probe's basestation. 

.. image:: ../../_static/images/plugins/nidaqmx/NPX_NIDAQSynchronized.png
  :alt: NIDAQmx plugin synced

.. tip:: For more information about recording and synchronization in Open Ephys, please see the :ref:`recordingdata` page.

NIDAQ Device Comparison
#########################################

We have tested the following devices, in descending order of performance. In all of the below examples, a 5 volt, ~2 Hz square wave @50% duty cycle from an Arduino is used to demonstrate the analog and digital input behaviors you can expect to see using each device in Open Ephys.

If you have any additional devices available for testing, please let us know!

PXI-6133 
---------
The PXI‑6133 supports simultaneous-sampling, which means the analog and digital channels are guaranteed to be sampled simultaneously in time and synchronized with each other before they reach the GUI. Passing the square wave test signal to both the first analog and first digital input on the card shows that both signals are synchronized:

.. image:: ../../_static/images/plugins/nidaqmx/6133_SyncedPulse_zoomed.png
  :alt: NIDAQmx plugin PXO-6133 synced input signal magnified

.. note:: The 6133 takes a few (3-5) seconds to initialize after the acquisition button has been pressed; therefore, there will be a delay before data starts to appear in the LFP Viewer.

Furthermore, the analog input channels on the 6133 are well-isolated and there is no signal cross-talk on any of the unused analog input channels:

.. image:: ../../_static/images/plugins/nidaqmx/6133_SyncedPulse.png
  :alt: NIDAQmx plugin PXI 6133 synced signal with no interference on unused channels

.. tip:: The voltage on the unused channels may float at a constant non-zero value if left open. It is good practice to either ground or short the non-used analog inputs to avoid this. 


PXIe-6341
----------
The PXIe-6341 performs almost as well as the 6133, however, there is cross-talk across unused channels when applying the same test signal as above. That is, if a signal is physically connected to only the first analog input channel, a ‘ghost’ of the signal will appear across all of the open analog input channels as shown below:

.. image:: ../../_static/images/plugins/nidaqmx/6341_SyncedPulse.png
  :alt: NIDAQmx plugin PXIE-6341 syned input signal

However, if a 10Hz square wave is added to the second analog input and to the second digital input, there is no interference between the new signal and the original test signal on either the analog or digital channels:

.. image:: ../../_static/images/plugins/nidaqmx/6341_Combo.png
  :alt: NIDAQmx plugin PXIE-6341 multiple analog and digital inputs without interference

PXI-6521 
----------
The 6521 is the least expensive PXI DAQ board available from NI. However, it only supports digital input channels that swing higher than +11V in order to get a TTL signal. Nonetheless, this card can still be a cheap(er) alternative for experiments that may only require digital channels alongside neural data. If you are using a 5V microcontroller in your experiments, you will need to boost your digital lines from 5V to something higher than 11V. A potential solution here is to use a 12V power source and a transistor inverter circuit.

PCIe-6321
-----------
The PCIe-6321 card does not require a PXI chassis and can be connected directly to the PCIe slot of a PC motherboard. This card exhibits the same behavior with the test signal as the PXI-6341, however, there is some overshooting observed at steep signal transition points. Smooth analog signals will not exhibit this behavior. 

.. image:: ../../_static/images/plugins/nidaqmx/6321_synced_ringing.png
  :alt: NIDAQmx plugin PCIE-6321 seep signal ringing

USB-6001
----------
Applying the test signal to the first analog and second digital input of the USB 6001 results in the digital channels going high significantly before the analog channels. The USB devices also inherit the ghosting and overshooting issues mentioned in previous devices. 

.. image:: ../../_static/images/plugins/nidaqmx/USB-6001-Unsynced.png
  :alt: NIDAQmx plugin USB-6001 ghosting with single analog channel

The delay between the observed analog and digital inputs is not guaranteed to be constant. By grounding any unused analog channels, the USB-6001 can still achieve a clean and isolated signal on its analog channels:

.. image:: ../../_static/images/plugins/nidaqmx/USB-6001-Grounded.png
  :alt: NIDAQmx plugin USB-6001 grounded unused analog inputs
 