.. _kwikdataformat:
.. role:: raw-html-m2r(raw)
   :format: html

KWIK format
=====================

.. caution:: The KWIK format has been deprecated, and is not recommended for use with the Open Ephys GUI

The complete specification (created by Ken Harris' lab at UCL) can be found `here <https://github.com/klusta-team/kwiklib/wiki/Kwik-format>`__. Open Ephys uses a variation on this specification for two reasons:

#. The KWIK file is overwritten by KlustaKwik, so we save our event-related metadata in a separate file (the KWE file)

#. It's possible to save data from multiple processors simultaneously, so we create a separate KWD (continuous) file for each processor

Each recording will generate 3 or more files:

* :code:`experiment1_XXX.raw.kwd` contains the continuous data for a given processor (:code:`#XXX`) in the /recordings/#/data dataset, where # is the recording number, starting at 0

* :code:`experiment1.kwx` contains the spike waveforms

* :code:`experiment1.kwe` contains the event data (this is an Open Ephys-specific file)

Some simple Python functions for loading KWIK datasets can be found in the `analysis-tools <https://github.com/open-ephys/analysis-tools>`__ repository on GitHub.

* load(kwd_filename) – imports a continuous dataset into a Python dictionary

* get_rising_edge_times(kwe_filename, TTLchan) – extracts the rising edge ("ON") events for a given TTL channel

* get_falling_edge_times(kwe_filename, TTLchan) – extracts the falling edge ("OFF") events for a given TTL channel

* get_experiment_start_time(kwe_filename) – extracts the offset between the event times and the timestamps loaded from the KWD file

