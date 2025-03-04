.. _meanspikerate:
.. role:: raw-html-m2r(raw)
   :format: html

#####################
Mean Spike Rate
#####################

.. image:: ../../_static/images/plugins/meanspikerate/meanspikerate.png
  :alt: Annotated Mean Spike Rate editor

.. csv-table:: Estimates the mean spike rate across time and channels. Uses an exponentially weighted moving average to estimate a temporal mean (with adjustable time constant), and averages the rate across selected spike channels (electrodes). Outputs the resulting rate onto a selected continuous channel (overwriting its contents).
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Mac, Linux"
   "*Built in?*", "No"
   "*Key Developers*", "Ethan Blackwood"
   "*Source Code*", "https://github.com/open-ephys-plugins/mean-spike-rate"

Installing and upgrading
###########################

The Mean Spike Rate plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to access the Plugin Installer, browse to the "Mean Spike Rate" plugin, and click the "Install" button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.


Plugin configuration
######################

The plugin editor allows the user to set the following parameters:

- **Active Spike Channels:** The spike channels to include in the mean spike rate calculation.

- **Spike Channel Scroll Bar:** Scrolls through the list of incoming spike channels for the active input stream.

- **Output Channel:** The continuous channel to output the mean spike rate onto. The mean spike rate will overwrite the contents of this channel.

- **Time Constant** (ms): The time constant of the exponentially weighted moving average used to estimate the mean spike rate.