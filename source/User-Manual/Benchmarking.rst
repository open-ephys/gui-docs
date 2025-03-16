.. _benchmarking:
.. role:: raw-html-m2r(raw)
   :format: html

Benchmarking
=====================

To better understand the limits of the Open Ephys GUI, the :ref:`sourcesim` plugin can be used to vary the number of data channels and input streams. The plugin generates synthetic data that mimics the output of a real acquisition system, allowing users to test the GUI's ability to process, visualize, and record data in real time.

As the number of input channels increases, there are two limits that may be reached:

- **Available processing time:** The fraction of time spent processing each incoming data buffer may exceed the overall buffer duration (default: ~20 ms), potentially resulting in dropped samples. This is indicated by the CPU monitor in the GUI's control panel. The CPU monitor should remain well below 100% at all times to ensure data integrity.

- **Disk writing speed:** The Record Node buffers each data stream before transferring samples to disk, to prevent the time spent writing data from delaying data processing by downstream plugins. If the disk writing speed is slower than the rate of incoming data, the Record Node will eventually run out of buffer space. This is indicated by the buffer monitor associated with each stream in a Record Node. If this buffer becomes more than 90% full, the Record Node will stop data acquisition to ensure that all files can be closed gracefully.

Summary of results
-------------------

.. note:: The exact performance of the GUI will depend on the your computer specs and your signal chain. You can use the :ref:`sourcesim` plugin to test the maximum number of channels/streams that can be comfortably acquired on your system.

On a powerful workstation (Windows 10, 32-core Intel Core i9, 128 GB RAM, and three NVMe SSDs), the GUI can record the following types of inputs without issues:

* **32 Neuropixels 1.0 probes** (writing to three SSDs)
* **32 Neuropixels 2.0 probes** (writing to two SSDs)
* **5760 channels** from a single stream (writing to one SSD)

For a standard laptop (Windows 10, 16-core Intel Core i7, 32 GB RAM, and one internal SSD), the following configurations are possible:

* **8 Neuropixels 1.0 probes**
* **8 Neuropixels 2.0 probes**
* **4000 channels** from a single stream


Detailed results
----------------

The following tables show the results of benchmarking the GUI under a variety of conditions.


Workstation
############

| **Computer:** Windows 10, 32-core Intel Core i9, 128 GB RAM, 3 NVMe SSDs
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 40 ms
| **Simulated source:** Neuropixels 1.0

.. csv-table::
   :widths: 30, 30, 30, 30, 30, 30, 70

   "**Probes**", "**SSD #1**", "**SSD #2**", "**SSD #3**", "**CPU**", "**RAM**", "**Notes**"
   "12", "12", "-", "-", "9%", "13.5 GB", " "
   "16", "16", "-", "-", "11%", "18 GB", "Record buffer overflow"
   "24", "12", "12", "-", "16%", "27 GB", " "
   "30", "15", "15", "-", "18%", "57.5 GB", " "
   "32", "12", "10", "10", "19%", "61.5 GB", " "

|

| **Computer:** Windows 10, 32-core Intel Core i9, 128 GB RAM, 3 NVMe SSDs
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 40 ms
| **Simulated source:** Neuropixels 2.0

.. csv-table::
   :widths: 30, 30, 30, 30, 30, 70

   "**Probes**", "**SSD #1**", "**SSD #2**", "**CPU**", "**RAM**", "**Notes**"
   "16", "16", "-", "9%", "16 GB", " "
   "20", "20", "-", "10%", "20 GB", "Record buffer overflow"
   "24", "12", "12", "13%", "24 GB", " "
   "32", "16", "16", "16%", "32 GB", " "
   "36", "18", "18", "18%", "36 GB", "Processing monitor reaches 100%"


|

| **Computer:** Windows 10, 32-core Intel Core i9, 128 GB RAM, 3 NVMe SSDs
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 40 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "2480", "10%", "7 GB", " "
   "3840", "12%", "11 GB", " "
   "4608", "13%", "12 GB", " "
   "5376", "15%", "14 GB", " "
   "5760", "15%", "15.5 GB", " "
   "6144", "15%", "16 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 32-core Intel Core i9, 128 GB RAM, 3 NVMe SSDs
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`BandpassFilter` > :ref:`LFP Viewer`
| **Buffer duration:** 40 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "2048", "12%", "7 GB", " "
   "3840", "16.4%", "11 GB", " "
   "4608", "18%", "12.4 GB", " "
   "5376", "20%", "14.4 GB", " "
   "5760", "20%", "15.5 GB", " "
   "6144", "23%", "16 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 32-core Intel Core i9, 128 GB RAM, 3 NVMe SSDs
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`BandpassFilter` > :ref:`LFP Viewer` > :ref:`spikedetector` > :ref:`spikeviewer`
| **Buffer duration:** 40 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "512", "12%", "7 GB", " "
   "768", "16.4%", "11 GB", " "
   "832", "18%", "12.4 GB", " "
   "896", "20%", "14.4 GB", "Processing monitor reaches 100%"



Laptop
############

| **Computer:** Windows 10, 16-core Intel Core i7, 32 GB RAM, SKHynix 512 GB SSD
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 21 ms
| **Simulated source:** Neuropixels 1.0

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Probes**", "**CPU**", "**RAM**", "**Notes**"
   "8", "16.5%", "8 GB", " "
   "9", "17%", "8.5 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 16-core Intel Core i7, 32 GB RAM, SKHynix 512 GB SSD
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 21 ms
| **Simulated source:** Neuropixels 2.0

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Probes**", "**CPU**", "**RAM**", "**Notes**"
   "8", "13.6%", "4.8 GB", " "
   "9", "14%", "5.4 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 16-core Intel Core i7, 32 GB RAM, SKHynix 512 GB SSD
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`LFP Viewer`
| **Buffer duration:** 21 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "2000", "19.4%", "3.5 GB", " "
   "2500", "22%", "4.3 GB", " "
   "3000", "23%", "5.1 GB", " "
   "4000", "25.8%", "6.8 GB", " "
   "4500", "29.4%", "7.5 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 16-core Intel Core i7, 32 GB RAM, SKHynix 512 GB SSD
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`BandpassFilter` > :ref:`LFP Viewer`
| **Buffer duration:** 21 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "2000", "24.6%", "3.5 GB", " "
   "2500", "30%", "4.3 GB", " "
   "3000", "31.2%", "5.1 GB", " "
   "4000", "39%", "6.8 GB", " "
   "4500", "43%", "7.5 GB", "Record buffer overflow"

|

| **Computer:** Windows 10, 16-core Intel Core i7, 32 GB RAM, SKHynix 512 GB SSD
| **Signal chain:** :ref:`sourcesim` > :ref:`Record Node` > :ref:`BandpassFilter` > :ref:`LFP Viewer` > :ref:`spikedetector` > :ref:`spikeviewer`
| **Buffer duration:** 21 ms
| **Simulated source:** Single stream (e.g. Open Ephys Acquisition Board)

.. csv-table::
   :widths: 30, 30, 30, 70

   "**Channels**", "**CPU**", "**RAM**", "**Notes**"
   "512", "20.7%", "1 GB", " "
   "640", "21.7%", "1.2 GB", " "
   "768", "22%", "1.4 GB", "Processing monitor reaches 100%"

|