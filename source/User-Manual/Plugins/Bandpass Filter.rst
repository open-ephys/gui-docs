.. _bandpassfilter:
.. role:: raw-html-m2r(raw)
   :format: html

################
Bandpass Filter
################

.. image:: ../../_static/images/plugins/bandpassfilter/bandpassfilter-01.png
  :alt: Annotated Bandpass Filter settings interface

|

.. csv-table:: Filters incoming continuous data between the specified low cutoff and high cutoff frequencies (Hz) using a 2nd-order Butterworth filter.
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/master/Plugins/FilterNode"

Plugin configuration
=====================

Individual channels can be set to have different parameters by highlighting them in the "PARAMS" tab in the hidden drawer on the right side of the plugin editor. The editor displays the parameters of the most recently selected channel(s).

.. caution:: Occasionally, the filter settings will only update for every other channel. If this happens, remove the Bandpass Filter from the signal chain and drag another one back in.


Filter specs
==============

To obtain the same filter in MATLAB, you can use:

.. code-block:: matlab

  [b, a] = butter(2, [lowcut highcut]/(samplerate/2));

  y = filter(b, a, x); % only filters in the forward direction

where :code:`lowcut` and :code:`highcut` are low and high cutoff frequencies (Hz), :code:`samplerate` is the sampling frequency (Hz), :code:`x` is the unfiltered data, and :code:`y` is the filtered data.

This is correct despite the fact that MATLAB's documentation claims it should produce a 4th-order filter (it uses a different convention).


