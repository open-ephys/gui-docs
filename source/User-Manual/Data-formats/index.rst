.. _dataformats:

Data formats
===============

Data formats in the GUI are implemented as plugins. This means you can extend the supported formats by installing additional plugins through the Plugin Installer. By default, only the Binary format is available, but other formats such as Open Ephys and NWB can be added as needed.

The GUI currently supports the following data formats:

* :ref:`binaryformat` (default format) stores the data as an array of samples in the following order: ch1 sample1, ch2 sample 1, ch3 sample 1, ..., chN sample 1, ch1 sample 2, ch2 sample 2, etc. It's the most compact format, and therefore can be loaded very quickly. Extra data, like timestamps or events are stored in `NumPy`_ format, which can be easily read using standard libraries. Metadata about the recording, such as sample rates or channel counts are stored in an easy-to-parse JSON file.

* :ref:`openephysformat` (available via the Plugin Installer) stores data in files that contains both the data samples and markers that allow data to be recovered if any part of the file becomes corrupted. Each channel is stored in a separate file, which can be convenient or inconvenient for analysis, depending on what you need to do. It's the first format that was used by the GUI, so it's been around for the longest amount of time. However, it's specific to Open Ephys, and will likely need to be converted to be compatible with most analysis tools.

* :ref:`nwbdataformat` (available via the Plugin Installer) stores data inside an HDF5 file, according to the specification defined by the `Neurodata Without Borders`_ project. It's based on a one-file-per-experiment philosophy, and aims to ensure that the data is self-documenting and easy to share. It comes with all the baggage of HDF5, including the need for a large library to access the data, and the inability to read any of the data if the file becomes corrupted. NWB files written by the GUI are compatible with the NWB 2.0 standard and can be read by the `pynwb`_ library.

.. _pynwb: https://pynwb.readthedocs.io/en/stable/
.. _NumPy: https://numpy.org/
.. _Neurodata Without Borders: https://nwb.org/


.. toctree::
    :hidden:
    :maxdepth: 5

    Binary-format
    Open-Ephys-format
    NWB-format

.. role:: raw-html-m2r(raw)
   :format: html
