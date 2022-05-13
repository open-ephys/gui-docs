.. _filesources:

.. default-domain:: cpp

File Sources
=====================

.. csv-table:: File Sources stream pre-recorded continuous data and events into the File Reader.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::FILE_SOURCE`"
   "*Base Class*", ":code:`FileSource`"
   "*Template*", "https://github.com/open-ephys-plugins/file-source-template"

A File Source plugin allows the GUI's File Reader to stream data from a new type of data format. By default, the File Reader can load data stored in the GUI's :ref:`binaryformat`. Plugins for the :ref:`openephysformat` and :ref:`nwbformat` can be installed via the Plugin Installer.

A data format may contain data for multiple recordings and/or streams, but the File Reader can only read one at a time. To read in multiple data streams simultaneously, multiple File Readers can be connected via a :ref:`merger`.

All file Source plugins must implement the following methods:

.. function:: bool open(File file)

   When this method is called, the File Source should attempt to open the file selected by the user, and return :code:`true` if successful. If the file is not valid or cannot be read for any reason, this method should return :code:`false`. For formats that distribute data across multiple files, this file should contain configuration information that can be used to access all of the additional files. For example, in the :ref:`binaryformat`, this is a JSON file an :code:`.oebin` extension.

   :param file: A Juce :code:`File` object representing the input file. The plugin can call :code:`File::getFullPathName()` to return a string containing the complete path to this file.


.. function:: fillRecordInfo()

   Instructs the File Source plugin to fill the :code:`infoArray` and :code:`eventInfoArray` objects with the relevant information for all recordings.


.. function:: updateActiveRecord(int index)

   Update the recording to be read in


.. function:: seekTo(int64 sample)

   Seek to a specific sample number within the active recording


.. function:: readData(int16* buffer, int nSamples)

   Read in nSamples of int16 data into a temporary buffer


.. function::  processChannelData(int16* inBuffer, float* outBuffer, int channel, int64 nSamples)

   Convert nSamples of data from int16 to float


.. function::  processEventData(EventInfo& info, int64 startTimestamp, int64 stopTimestamp)

   Add info about events occurring within a sample range
