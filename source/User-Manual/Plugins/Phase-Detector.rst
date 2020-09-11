.. _phasedetector:
.. role:: raw-html-m2r(raw)
   :format: html

################
Phase Detector
################

.. image:: ../../_static/images/plugins/phasedetector/phasedetector-01.png
  :alt: Phase Detector plugin settings interface

|

.. csv-table:: Emits events when a continuous signal arrives at a specific "phase," which really means a peak, falling zero-crossing, trough, or rising zero-crossing.
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/master/Plugins/PhaseDetector"

.. note:: Compare to Phase Calculator

Plugin configuration
######################

To use it, you have to select a phase (one of four buttons on the sine wave), an input channel (the continuous channel used for triggering), and an output channel (an event channel). Optionally, you can also select a "gate" channel, which causes the Phase Detector to emit its output only when that event channel is active (i.e., "high").

You can add additional detectors that target different phases or different channels by clicking the + button.

All continuous channels pass through the Phase Detector unchanged.

.. note:: Unless it follows a relatively narrow :ref:`bandpassfilter`, the Phase Detector will emit events at very unpredictable intervals, and probably won't be that useful. It's meant for detecting the phase of an oscillation in a particular frequency range, such as the theta band.



