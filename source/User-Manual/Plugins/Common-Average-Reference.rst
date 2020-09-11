.. _commonaveragereference:
.. role:: raw-html-m2r(raw)
   :format: html

#########################
Common Average Reference
#########################

.. image:: ../../_static/images/plugins/commonaveragereference/commonaveragereference-01.png
  :alt: Annotated Common Average Reference settings interface

|

.. csv-table:: Used to create a reference from the average of multiple channels. Helpful for removing noise that's shared across many electrodes.
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Ryan Maloney, Josh Siegle, Kirill Abramov"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/master/Plugins/CAR"

Plugin configuration
=====================

Use the **REFERENCE | AFFECTED** switch to choose which set of channels are being selected. Then, select the appropriate channels using the interface in the "PARAM" tab of the channel drawer on the right side of the plugin. It's possible to use Matlab-like range selection syntax to speed up the selection process.

The affected channels will have the average of all reference channels subtracted from their incoming signals. The gain slider adjusts the percentage of the average subtracted from the affected channels data.

