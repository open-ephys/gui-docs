.. _usermanual:

User Manual
===============

Welcome to the Open Ephys GUI user manual! Here you will find information about how to run experiments with this software. If you're interested in learning how to write your own plugins or better understand the GUI's source code, check out the :ref:`developerguide`.

The GUI was designed to process, record, and visualize multichannel extracellular electrophysiology data. It currently includes plugins that are compatible the following data sources:

* `Open Ephys acquisition board <https://open-ephys.org/acq-board>`__
* `Intan RHD USB Interface Board <https://intantech.com/RHD_USB_interface_board.html>`__
* `Intan RHD Recording Controller <https://intantech.com/recording_controller.html>`__
* `Neuropixels probes <https://www.neuropixels.org/>`__
* `XDAQ ONE and Core <https://www.kontex.io/xdaq>`__
* `EEG headsets that use Lab Streaming Layer <https://labstreaminglayer.org/#/>`__

This manual covers key topics, such as:

* :ref:`exploringtheui`

* :ref:`buildingasignalchain`

* :ref:`recordingdata`

It also includes documentation for individual :ref:`plugins`, the heart of the GUI's functionality.

If there's something missing from this documentation that you think should be included, please `open an issue on GitHub <https://github.com/open-ephys/gui-docs/issues>`__ or send a message to the `Open Ephys forum <https://groups.google.com/forum/#!forum/open-ephys>`__.

.. toctree::
    :hidden:
    :maxdepth: 1

    Before-you-begin
    Hardware-requirements
    Installing-the-GUI
    Exploring-the-user-interface
    Building-a-signal-chain
    Recording-data/index
    Remote-control
    Benchmarking
    Whats-new-in-v06x
    Whats-new-in-v100
    Plugins/index

.. role:: raw-html-m2r(raw)
   :format: html
