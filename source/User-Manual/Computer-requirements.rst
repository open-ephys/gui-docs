.. _computerrequirements:
.. role:: raw-html-m2r(raw)
   :format: html

Computer requirements
======================

If you are planning to run experiments with the Open Ephys GUI, you'll need to make sure your system is capable enough to keep up with data acquisition. In general, the minimum requirements will depend on how many channels you'd like to acquire simultaneously, and whether you need low-latency closed-loop feedback. Below, we provide some guidelines for configuring your data acquisition computer.

.. important:: The most important thing you can do to improve performance is use a solid-state drive (ideally NVMe) for writing data. And if you need to record ≥128 channels simultaneously, an SSD is required.

Operating system
#####################

The majority of users run the GUI on **Windows 10 or 11**, but it performs equally well on **macOS** and **Linux**. Almost all plugins are available for all three operating systems, with the exception of the :ref:`neuropixelspxi` and :ref:`onebox` plugins, which are only available for Windows, due to limitations of the API provided by IMEC. 

Processor speed
#####################

A **4-core, 3.0+ GHz** processor will work for basic signal chain configurations. Having more cores and/or a faster clock speed will allow you to build more complex signal chains without worrying about CPU overload. If the CPU monitor in the GUI's control panel is consistently above 50%, it means your computer is having trouble keeping up with the data processing requirements.

Memory
#####################

At least **1 GB of RAM per 32 channels** is required. For example, if you are acquiring 128 channels, you should have at least 4 GB of RAM. For 512 channels, you should have at least 16 GB of RAM.

Data storage
#####################

A **solid state drive** for writing data is *strongly* recommended for all configurations, and required for any recordings involving at least 128 channels. For recordings with more than 2000 channels, it may be necessary to record data across two or more SSDs in parallel by placing multiple Record Nodes in the signal chain. If the recording buffer on a Record Node starts filling up, it means your computer can't write the data as quickly as it is being generated.

Graphics card
###############

As of version 1.0, the GUI uses a GPU renderer by default, so having a powerful graphics card can help for visualization. If you do not have access to a GPU, you can switch to a CPU renderer.


|
