.. _filesources:

.. default-domain:: cpp

File Sources
=====================

.. csv-table:: File Sources stream pre-recorded continuous data and events into the File Reader.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::FILE_SOURCE`"
   "*Base Class*", ":code:`FileSource`"
   "*Template*", "https://github.com/open-ephys-plugins/file-source-template"

A File Source plugin allows the GUI's File Reader to stream data from a new type of data format. By default, the File Reader can load data stored in the GUI's :ref:`binaryformat`. File Source plugins for the :ref:`openephysformat` and :ref:`nwbdataformat` can be installed via the Plugin Installer.

A data format may contain data for multiple recordings and/or data streams, but the File Reader can only read one of these at a time. Each "Record" within a File Source represents one recording (a block of contiguous samples) from one data stream (a block of channels that are sampled synchronously). When a new file is loaded, the File Source must provide information about all of the Records that are available, and then the user can select which one to read at any given time. To read in data from multiple recordings or data streams simultaneously, multiple File Readers can be connected via a :ref:`merger`.

To create a new File Source, start with the `File Source Template <https://github.com/open-ephys-plugins/file-source-template>`__ and add code that implements the following methods:

.. function:: bool open(File file)

   When this method is called, the File Source should attempt to open the file selected by the user, and return :code:`true` if successful. If the file is not valid or cannot be read for any reason, this method should return :code:`false`. For formats that distribute data across multiple files, this file should contain configuration information that can be used to access all of the additional files. For example, in the :ref:`binaryformat`, this is a JSON file with an :code:`.oebin` extension.

   :param file: A Juce :code:`File` object representing the input file. The plugin can call :code:`File::getFullPathName()` to return a string containing the complete path to this file.


.. function:: void fillRecordInfo()

   Instructs the File Source to fill the :code:`infoArray` and :code:`eventInfoArray` objects with the relevant information for all recordings ("Records").


.. function:: void updateActiveRecord(int index)

   Informs the File Source that a new Record has been selected by the user.

   :param index: The index of the selected Record within the :code:`infoArray` object.


.. function:: void seekTo(int64 sample)

   Informs the File Source about the next sample index that should be read from the active Record.

   :param sample: The 0-based sample index to seek to.


.. function:: void readData(int16* buffer, int nSamples)

   Reads data from the active Record into a temporary integer buffer.

   :param buffer: A buffer of 16-bit integers that will hold the incoming data.
   :param nSamples: The total number of samples to read.


.. function:: void processChannelData(int16* inputBuffer, float* outputBuffer, int channel, int64 nSamples)

   Converts data from integers to floats so it can be processed by downstream plugins.

   :param inputBuffer: A temporary buffer of unscaled 16-bit integers.
   :param outputBuffer: A buffer of floats containing data in real-world values (usually microvolts).
   :param channel: The index of the channel to convert.
   :param nSamples: The total number of samples to convert.


.. function:: void processEventData(EventInfo& info, int64 startSampleNumber, int64 stopSampleNumber)

   Adds info about the events that occurred within a given range of samples.

   :param info: The :code:`EventInfo` object to be updated.
   :param startSampleNumber: The minimum sample at which the incoming events can occur.
   :param stopSampleNumber: The maximum sample at which the incoming events can occur.
