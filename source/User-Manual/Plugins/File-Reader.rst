.. _filereader:
.. role:: raw-html-m2r(raw)
   :format: html

File Reader
=====================

.. image:: ../../_static/images/plugins/filereader/filereader-01.png
  :alt: Annotated File Reader settings interface

.. csv-table:: Reads data from a local file.
   :widths: 18, 80

   "*Plugin Type*", "Source"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Aarón Cuevas López, Pavel Kulik"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/main/Source/Processors/FileReader"

Loading Data
###################

By default, the File Reader is compatible with files saved in the :ref:`binaryformat` (indexed with a :code:`structure.oebin` file). If the appropriate plugins are installed, the File Reader can also read data stored in the :ref:`openephysformat` (indexed with a :code:`structure.openephys` file) and :ref:`nwbdataformat` (saved inside a :code:`*.nwb` file). 

.. note:: Data saved by previous versions of the GUI may not load in version 1.0.x. For help updating your files to work with the latest File Reader, send a message to `support@open-ephys.org <mailto:support@open-ephys.org>`__.

Each input file can contain multiple streams/recordings, but only one stream/recording can be played back per File Reader during acquisition. The drop-down menu below the active file name makes it possible to switch between the available streams when acquisition is paused. In order to play multiple streams simultaneously, you can merge multiple File Readers, each with the same input file and a different selected stream.  

You can play back a subset of a recorded file by changing the "start" and "end" times at the bottom of the File Reader editor.

File Scrubbing
######################

The drawer button on the left side of the File Reader opens the Scrubbing Interface. This interface makes it possible to quickly browse to different parts of the data file.

**Full Timeline**

The bottom (full) timeline spans the length of the recording between the start and end times as set in the File Reader editor. The colored vertical bars along the timeline correspond to recorded events associated with the active stream. For recordings longer than 30 seconds, the colored slider along this timeline is fixed at a 30 second interval and allows adjusting the start and stop time for playback.

On the bottom left of the Full Timeline, there are two time values. The bottom darker value is the absolute start time of the recording. The top lighter value is the same as the currently selected start time in the File Reader editor. 

The middle time value is the time of the current playback position. 

On the bottom right of the Full Timeline, the bottom time value is the absolute end of the recording. The upper, lighter value is the same as the currently selected end time in the File Reader editor.

**Zoom (30 Second) Timeline**

The top timeline spans the currently selected interval of 30 seconds as set in the Full Timeline. The Zoom Timeline contains a playback position slider that can be moved to any point along the Zoom Timeline. Dragging and releasing the slider will move the playback position to the new position both during acquisition and while acquisition is paused. 

The top left time value is the start of the 30-second interval. The middle value is the current slider position time. The top right time value is the end of the 30-second interval.

Sample Data
######################

By default, the File Reader reads a recording of 16 channels from 8 stereotrodes implanted in the barrel cortex of an awake mouse.

There are also four example files included in the `GUI's GitHub repository <https://github.com/open-ephys/plugin-GUI>`__ that can be used with the File Reader, found in the :code:`Resources/DataFiles` directory.

* :code:`data_stream_16ch_cortex` - Same as the default data file. The signals contain many action potentials, and are useful for testing spike detection pipelines.

* :code:`data_stream_16ch_hippocampus` - One channel of data recorded from mouse CA1, replicated across 16 channels. The signals have large-amplitude theta oscillations, and can be used to test phase-triggered stimulation.

* :code:`data_stream_sine_wave` - 16 channels of a 1000-microvolt sine wave.

* :code:`chirps_16_channels_At40kHz` - 16 channels of frequency sweeps at 40 kHz.

Other input data
######################

The File Reader should be able to read any datasets that have been previously saved by the GUI (v0.5.0+). If you make a recording that is not compatible with the File Reader for some reason, please send a message to :code:`support@open-ephys.org` for conversion assistance.