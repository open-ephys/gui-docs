.. _synchronization:

.. role:: raw-html-m2r(raw)
   :format: html


Synchronizing Data Streams
============================

The Open Ephys GUI is able to acquire, process, and save data from multiple asynchronous data streams simultaneously. However, even if two data streams have identical sample rates, they are neither guaranteed to start acquisition simultaneously nor acquire data at exactly the advertised sample rate. Therefore, some synchronization procedure is required. 

Synchronization is typically performed offline, using the timing events on a "sync line" that is shared between each data acquisition device. But the GUI can also be configured to synchronize data from separate streams in real time, as it is being written to disk. This tutorial uses the example of Neuropixels probes and a National Instruments Data Acquisition (NIDAQ) device to demonstrate how to synchronize data streams online.

General principles of synchronization
=======================================
 
Whether synchronization is being performed online or offline, all data streams must share a common hardware sync line. Synchronization in software is accurate to within a few milliseconds, but is not precise enough for synchronizing electophysiogical signals acquired at tens of kilohertz.

The following diagram demonstrates the basic logic behind hardware-level synchronization:

.. image:: ../_static/images/tutorials/synchronization/sync-overview-01.png
  :align: center
  :alt: Hardware synchronization diagram

There are two data streams, *A* and *B*, which have approximately the same sample rate. However, they start at slightly different times, and the clocks drift apart over the course of the recording. Therefore, it's impossible to know which samples occurred simultaneously without comparing them to a reference signal that's shared across the two data streams

In this example, there are a few sample numbers that are important to note:

- The samples at which the recordings were stopped and started (**10** and **114** for *Stream A*, and **25** and **127** for *Stream B*).
- The samples at which the first recorded low-to-high transition on the sync line occurred (**112** for *Stream A* and **27** for *Stream B*).
- The samples at which the last recorded low-to-high transition on the sync line occurred (**27** for *Stream A* and **125** for *Stream B*).

With this information in hand, we can translate all of the timestamps from *Stream A* into timestamps from *Stream B*, and vice versa.

First, we'll compute a scaling factor, which is the ratio of the total number of samples between the first and last sync transitions:

.. code::

  t_first_A = 12
  t_last_A = 112

  t_first_B = 27
  t_last_B = 125

  scaling = (t_last_A - t_first_A) / (t_last_B - t_first_B)

In this case, the scaling factor is equal to :code:`100 / 98`, or 1.0204.

Now, we can translate all the timestamps from *Stream B* into timestamps from *Stream A*:

.. code::

  timestamps_B = range(25, 127)
  timestamps_A = (timestamps_B - t_first_B) * scaling + t_first_A

We can check this by plugging in the value of the last sync transition time on *Stream B*: :code:`(125 - 127) * 1.02 + 12 = 111.96`. Rounded to the nearest integer, this is **112**, which is the actual time of the last sync transition on *Stream A*.

Understanding how to perform this basic procedure is extremely useful, since it will allow you to synchronize any data streams, even ones that are recorded by different data acquisition systems. As long as they share one digital input line, they can be synchronized offline.

The Open Ephys GUI can perform these calculations in real time, provided the hardware and software are set up correctly. However, it's still important to know what's going on under the hood, in case you need to troubleshoot your setup.

Hardware Configuration
######################

When using the :ref:`neuropixelspxi` and :ref:`nidaqmx` plugins, there are two options for setting up the sync line:

#. Use the built-in sync output of the Neuropixels PXI basestation.

#. Use a separate device, such as an Arduino, to generate a sync signal that is routed to the Neuropixels PXI basestation and the NIDAQ device.

In the first case, the Neuropixels’ basestation can be configured to output its own physical clock signal that can be output as an input into the NIDAQ device:

#. Connect one end of an SMA cable to the SMA connector at the bottom of the Neuropixels basestation.

#. Connect the other end of the SMA cable to a digital input channel of the NIDAQ device*. 

.. image:: ../_static/images/tutorials/synchronization/config_1.png
  :align: center
  :alt: NPX + NIDAQ 

The advantage of this option is that it doesn't require any additional devices. However, the Neuropixels basestation can only generate pulses at regular intervals (1 Hz or 10 Hz), which can make it ambiguous which pulses were the first and last, especially if you're synchronizing data streams outside of the Open Ephys GUI.

In the second case, the Neuropixels’ basestation can be configured as an input accepting the digital output of an Arduino, for example:

#. Connect one end of an SMA cable to the SMA connector at the bottom of the Neuropixels basestation.

#. Connect the other end of the SMA cable to a digital output channel of the Arduino.

#. Connect the same digital output of the Arduino into a digital input channel of the NIDAQ device.

.. image:: ../_static/images/tutorials/synchronization/config_2.png
  :align: center
  :alt: NPX + NIDAQ + Arduino

.. note:: * You will likely need an adapter to match the digital terminals.

The Arduino can be configured to generate sync pulses at pseudo-random intervals, which makes it possible to align data streams in an unambiguous way, even if they are stopped at started at different times.

For the purposes of this tutorial, either configuration will work.

Software Configuration
######################

Online synchronization only occurs within a the Open Ephys GUI's RecordNode as data is written to disk. This means that data coming into and out of a RecordNode in a signal chain is not necessarily synchronized. In order to synchronize online, the RecordNode must be configured to match the active hardware configuration: 

#. Download the Neuropixels-PXI and NIDAQmx source processors via "File > Plugin Installer".

#. Insert a Neuropixels-PXI source processor into the signal chain.

#. If using the Neuropixels-PXI to generate the sync pulses (option 1 above), change the default selection on the sync control pull-down menu from :code:`INPUT` to :code:`OUTPUT`. Use the default clock rate of 1 Hz.

#. Insert a NIDAQmx source processor into the signal chain (it will automatically create a new branch).

#. Select the Neuropixels-PXI processor in the signal chain and insert a Merger processor directly after it.

#. Right click on the title bar of the Merger and select "NIDAQmx" as the source processor to merge with.

#. Insert a RecordNode after the merger.

#. Select the ||| on the left side of the RecordNode to access the stream buffer monitors. The right most buffer monitor represents the NIDAQ stream, and any remaining buffers to the left represent the Neuropixels streams (two buffers at 30 kHz and 2.5 kHz for each 1.0 probe, one buffer at 30 kHz for each 2.0 probe).

#. Under each buffer monitor, click on the sync line monitor to select the digital input channel which matches the physical sync channel used in your hardware configuration. For Neuropixels there is only one channel available so it is automatically selected. For NIDAQ devices, there will likely be multiple digital channels available; select the channel used in the hardware that is connected to your sync signal.

#. Designate one of the streams to be the master processor. By default this will be the 30 kHz band of the first probe detected.

#. Ensure the Binary Format is selected in the RecordNode, as this is currently the only format that supports online synchronization.

#. Ensure "Record Events" is enabled in the RecordNode.

.. image:: ../_static/images/tutorials/synchronization/sync-tutorial-01.png
  :align: center
  :alt: Record Node Syncing

Monitoring and Recording
########################

At this point, the GUI is configured to write synchronized data to disk. In order to acquire and record synchronized data:

#. Start data acquisition by pressing the Play button in the Control Panel. The sync monitors turn orange once acquisition starts and then green as each stream becomes synchronized.

#. Wait until all the orange sync monitors turn green. This generally happens instantaneously, however, in some cases it may take a few seconds to stabilize.

#. Start recording by pressing the Record button in the Control Panel. Data streams with green sync control monitors will be written to disk with synchronized timestamps.

.. image:: ../_static/images/tutorials/synchronization/sync-tutorial-02.png
  :align: center
  :alt: Record Node Synchronized

Loading and Processing
######################

First, read the `Binary Format Docs <https://open-ephys.github.io/gui-docs/User-Manual/Recording-data/Binary-format.html>`__.
Synchronized data streams written to disk will contain an additional :code:`synchronized_timestamps.npy` file alongside the :code:`timestamps.npy` file. 
The :code:`synchronized_timestamps.npy` file contains one float timestamp (in seconds) for every integer timestamp (in sample number) found in the corresponding :code:`timestamps.npy` file. The :code:`synchronized_timestamps.npy` file provides a common time base to which timestamps belonging to the corresponding stream are mapped to.  

Events detected in a synchronized stream will only save their timestamps as sample numbers since acquisition started. For example, an event that occurred at sample number :code:`405012` will correspond to the continuous sample that occurred at timestamp :code:`405012`, which can then be mapped back to the common time base timestamp in the :code:`synchronized_timestamps.npy` file. 

For spike data, the values in :code:`spike_times.npy` represent the sample index at which the spike occurred, relative to the beginning of the :code:`continuous.dat` file. To get the actual timestamp at which the spike occurred, you must add the first timestamp from :code:`timestamps.npy` in the corresponding continuous data stream the spike was detected in. Again, that timestamp can be mapped back to the common time base timestamp in the :code:`synchronized_timestamps.npy` file. 

More information regarding offline analysis can be found `here <https://github.com/open-ephys/open-ephys-python-tools/tree/main/open_ephys/analysis>`__ for Python tools and `here <https://github.com/open-ephys/open-ephys-matlab-tools/tree/main/open_ephys/analysis>`__ for Matlab tools.

Questions? 
###########

If anything is still unclear after reading this tutorial, please reach out to :code:`support@open-ephys.org`, we will respond directly and update the tutorial as needed. 