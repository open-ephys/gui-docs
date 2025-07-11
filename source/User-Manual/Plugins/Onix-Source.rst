.. _onixsource:
.. role:: raw-html-m2r(raw)
   :format: html

#####################
Onix Source
#####################

.. image:: ../../_static/images/plugins/onixsource/onixsource-01.png
  :alt: Annotated Onix Source Editor

.. csv-table:: Streams data from the `Onix <https://open-ephys.github.io/onix-docs/>`__ acquisition system, which is a high-performance, low-latency data acquisition system designed for neuroscience research.
  :widths: 18, 80

  "*Plugin Type*", "Source"
  "*Platforms*", "Windows"
  "*Built in?*", "No"
  "*Key Developers*", "Brandon Parks, Jon Newman, Aarón Cuevas López"
  "*Source Code*", "https://github.com/open-ephys-plugins/onix-source"


Installing and upgrading
===========================

The Onix Source plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P**
or **⌘P** to access the Plugin Installer, browse to the "Onix Source" plugin, and click the "Install"
button.

The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's
already installed.

Plugin Configuration
===========================

The Onix Source plugin allows you to stream data from the Onix acquisition system. This plugin
assumes that you have an Onix Breakout Board connected and powered on before adding the plugin to
the signal chain. To install a Breakout Board, refer to the `Onix Breakout Board documentation <https://open-ephys.github.io/onix-docs/Getting%20Started/index.html/>`__.

To configure the plugin:

1. Add the Onix Source plugin to your signal chain.
2. In the plugin editor, select the Onix headstage you want to stream from, specifying if the
   headstage is plugged into Port A or Port B.
3. Set the block read buffer size, which determines how much data is read from the Onix system at
   once. A larger buffer size can reduce CPU usage but may increase latency.
4. Set the port voltage, or allow the automated voltage discovery algorithm to determine the port
   voltage automatically by leaving the field empty (this automatically defaults to "Auto",
   indicating that the plugin will attempt to discover the port voltage).
5. Press "Connect" to establish a connection to the Onix system.
6. Once connected, acquisition can be started in the GUI's main control panel.
7. For further configuration, click the plugin banner (i.e., the gray or colored portion of the
   editor showing the name of the plugin) to open the canvas and adjust settings as needed.

   a. The canvas is structured as tabs, with the first tab showing the breakout board and its
      devices, with the following tabs populating with the specific headstages selected. 
   b. Note: if the canvas is not visible, you can toggle it by clicking the "Toggle Canvas" button
      in the plugin editor. 

Plugin Editor
================

The Onix Source plugin editor provides a user-friendly interface for configuring the plugin. The
editor includes:

- Options to select the headstage on a specific port.
- Status indicators for the connection to the Onix system on each port.
- Ability to choose the port voltage, or use the automated voltage discovery algorithm.
- A textbox to set the block read buffer size.
- Monitor memory usage during acquisition.

All options in the editor are able to be changed at any time before the "Connect" button is pressed.
Once the "Connect" button is pressed, the plugin will attempt to connect to the Onix system using
the selected settings. If the connection is successful, the editor settings will be locked, and the
only way to change them is to disconnect from the Onix system by pressing the "Disconnect" button.
Canvas options are able to be changed at any time, even after the "Connect" button is pressed, up
until acquisition is started in the GUI's main control panel. During acquisition, all settings in the canvas are locked, and the plugin will not allow any changes until acquisition is stopped.

Port Specific Configuration
############################

The Onix Source plugin allows you to configure each port (A or B) independently. You can select the
headstage connected to each port, set the port voltage, and monitor the connection status.
Port A and Port B can be configured to stream data from different headstages, allowing for
flexibility in experimental setups. Select the desired headstage from the dropdown menu
for each port, and adjust the port voltage if necessary.

.. note:: 
  The Onix Source plugin only supports Neuropixels headstages at the moment. Other headstages may be supported in future releases.

Port Voltage Configuration
-----------------------------

The port voltage can be set manually or left empty to allow the plugin to automatically discover the
port voltage. If set to "Auto", the plugin will attempt to determine the port voltage automatically,
which is useful for ensuring compatibility with different headstages. Note that the port voltage
will only be automatically discovered if the field shows "Auto". If you set a specific voltage, the
plugin will use that value instead. If a previously set port voltage is removed, the plugin will
default to "Auto", indicating that it will attempt to discover the port voltage automatically.

If automated port voltage discovery is enabled, the plugin will attempt to determine the port
voltage when "Connect" is pressed. A progress bar will be displayed over the main GUI window showing
the status of each port as it cycles through pre-determined voltage values attempting to find a
connected headstage. If a valid port voltage cannot be determined, an error message will
be displayed in a pop-up window.

.. note:: 
  The automated port voltage discovery algorithm is designed to work with the Onix acquisition
  system and may not be compatible with other systems. If you encounter issues, please refer to 
  the Onix documentation or contact support.

Once the Onix system is connected, the voltage that was used will be displayed in the plugin editor.
If the port voltage is set to "Auto", the plugin will display the discovered voltage after the
connection is established, or if the port voltage was set manually, it will display the
configured voltage.

Port Status Indicators
-----------------------------

To the left of each Port, there is a status indicator that shows the connection status of the Onix
system on that port. The status indicators can show the following states:

- Empty circle: No connection to the Onix system, port voltage is zero.
- Green circle: Power is on, but the headstage is not connected.
- Purple circle: Power is on, and the headstage is connected.

When the "Connect" button is pressed, the plugin will attempt to connect to the Onix system on the
selected port. If the connection is successful, the status indicator will change to purple, and the
port voltage will be updated as described above. If the connection fails, the status indicator will remain empty, and an error message will be displayed in a pop-up window.

Block Read Size Configuration
###############################

The block read size can be configured by entering a value in the "Block Read Size" field. This value
determines how much data is read from the Onix system at once. A larger block read size can reduce
CPU usage but may increase latency. A smaller block read size can provide lower latency, but it can
cause the memory monitor to fill up. The default value is 4096 samples, but this can be adjusted
based on your experimental needs.

The block read size can be set to any integer value, but this value must be greater than the size of
the largest frame received from the connected headstage. If the block read size is set to a value
smaller than the size of the largest frame, an error message will be displayed in a pop-up window
with the smallest value possible for the block read size. The plugin will not allow you to connect
to the Onix system until the block read size is set to a valid value.

Liboni Version
################

The Onix Source plugin uses the `liboni <https://open-ephys.github.io/ONI/v1.0/api/index.html>`__
library to communicate with the Onix acquisition system. The version of liboni used by the plugin is
displayed in the plugin editor. This version is important for compatibility with the Onix system and
for enabling support. This version may be updated in future releases of the plugin.

Memory Usage Monitor
#######################

The Onix Source plugin includes a memory usage monitor that displays the current memory usage of the
hardware buffer. This monitor is useful for tracking memory usage during acquisition and can help
identify potential performance issues, particularly related to the block read size. The memory usage
monitor is updated in real time during acquisition, and displays the current memory usage as a
logarithmic percentage of the total memory available for the hardware buffer. For example, if the
memory buffer is approximately 1% full, the status bar will be filled to approximately 15% of its
total height. This logarithmic scale is used to provide a more intuitive representation of memory
usage, as it allows for easier visualization of small changes in memory usage.

For a linear view of the memory usage, you can hover over the memory usage monitor, which will
display the current memory usage as a percentage of the total memory available for the hardware
buffer. This will only update during acquisition, when acquisition is stopped the memory usage
monitor will show a tooltip describing its use.

The memory usage is also saved as a stream in the Onix Source plugin, allowing you to
visualize memory usage over time in the GUI. This can be useful for identifying trends in memory
over longer recordings, and for diagnosing potential performance issues related to memory usage.
This data stream is always enabled, and cannot be disabled.

Plugin Canvas
================

The Onix Source plugin includes a canvas that allows you to visualize the connected headstages and
their devices, easily changing their configuration settings. The canvas is structured as tabs, with
the first tab showing the breakout board and its devices, and the following tabs populating with the
specific headstage(s) selected. Each tab is structured similarly, with the following elements:

**Hub Tabs**

- *Hub Name*: The tab name shows the name of the hub, which is the headstage name in the case of
  headstages, or Breakout Board in the case of the breakout board.
- *Hub Devices*: The devices connected to the hub are listed as tabs under the hub name.

**Device Tabs**

- *Device Name*: The tab name shows the name of the device.
- *Device Enabled Status*: A button that allows you to enable or disable the device. If the device is
  enabled, the button will be orange, and if it is disabled, the button will be gray.
  
  - Not all devices can be disabled.

- *Save Settings Button*: A button that allows you to save the current settings for the device.
  Clicking the button will open a dialog that allows you to save the settings to an XML file.
  
  - Not all devices have settings that can be saved.

- *Load Settings Button*: A button that allows you to load settings from an XML file. Clicking the
  button will open a dialog that allows you to select an XML file to load the settings from.

  - Not all devices have settings that can be loaded.

Breakout Board Configuration
###############################

The first tab in the canvas is the Breakout Board tab, which shows the connected devices on the
breakout board. The breakout board is the main hub for the Onix system, and it is where the headstages are
connected. Certain devices on the breakout board can be enabled or disabled, and their settings can
be saved and loaded. The following devices are available for configuration on the breakout board:

- *AuxiliaryIO*: This tab allows you to configure the Analog and Digital I/O on the breakout board.
- *Harp Sync Input*: This tab allows you to configure the Harp Sync Input on the breakout board.
- *Output Clock*: This tab allows you to configure the Output Clock on the breakout board.

AuxiliaryIO
--------------

The AuxiliaryIO tab allows you to configure the Analog and Digital I/O on the breakout board. The
Analog I/O can be used to stream analog data from external devices, and the Digital I/O can be used to
stream digital data from external devices, as well as to record digital events from button presses
on the breakout board.

Analog data is streamed as a separate data stream, and can be visualized using the "LFP Viewer"
plugin. There are twelve analog channels available, and all channels are always enabled to record data.
The analog data is streamed at 25 kHz.

Digital data is saved as events, and can be visualized using the "LFP Viewer" plugin. The digital
events are streamed at 25 kHz, and can be used to record button presses on the breakout board. The
first 8 digital channels record the digital inputs, and the last 6 digital channels record the
button presses.

.. note:: 
  Digital channels are pulled high by default if no connection is given to the digital input. Events are overlaid on data, meaning that if no connections are given to any digital inputs, there will be eight event overlays on the Analog data stream, potentially making it difficult to see Analog data. To avoid this, you can either connect the digital inputs to ground, or disable the event overlay in the LFP Viewer.

Neuropixels Headstage Configuration
######################################

Neuropixels headstages are configured in the canvas by selecting the Neuropixels headstage tab.
While there are multiple types of Neuropixels headstages, the configuration is similar for all of
them. The canvas will display the following elements:

- *Probe Tab(s)*: Each probe connected to the headstage will have its own tab, showing the probe
  name. Clicking on the tab will show the probe configuration options. For more information on
  configuring Neuropixels probes, refer to the Probe Configuration section below.
- *BNO055 Tab*: If the headstage has a BNO IMU, a tab will be displayed showing the BNO configuration
  options. For more information on configuring the BNO IMU, refer to the BNO Configuration section
  below.

Probe Configuration
----------------------

The Neuropixels probe configuration options are displayed in the probe tab. Each probe tab will
include a probe viewer, allowing you to visualize the probe layout and select the electrodes to
stream. Depending on the probe type, the following options, and a button to view the selected option
in the probe viewer, may be available:

- *AP Gain*: The gain for the AP channels. 
- *LFP Gain*: The gain for the LFP channels.
- *Reference*: The reference channel for the probe.
- *AP Filter Cut*: Whether or not to apply a filter to the AP channels.

Channel Constraints
^^^^^^^^^^^^^^^^^^^^^

For Neuropixels probes, there will always be 384 channels enabled across the entire probe.
Therefore, when enabling electrodes (either manually or using channel presets), some previously
enabled electrodes will be disabled. Additionally, if more than 384 electrodes are manually selected
to be enabled, only the last 384 channels will end up being enabled. It is therefore recommended to
always double-check that the correct electrodes are enabled.

In addition to the absolute number of channels, there are other restrictions in place regarding
which combinations of electrodes can be enabled at any given time. Each electrode is assigned a
particular channel number; across the entire probe, no two electrodes that share the same channel
can be simultaneously enabled.

Channel presets take this into account automatically and ensure that the rules are followed. When
manually enabling electrodes, the indexing logic is applied in the order that electrodes are
selected. If two (or more) electrodes are selected that share a channel value, the highest indexed
electrode is the only one that will be enabled.

Probe Viewer
^^^^^^^^^^^^^^^^^^^^^

The probe viewer will show the probe layout, with the shank(s) drawn and the electrodes displayed as
squares. Each electrode can be selected by clicking on it, or clicking and dragging to select
multiple electrodes. The selected electrodes will be highlighted, and can be enabled by clicking the
"Select" button under the *Electrodes* label to the right of the probe viewer. There are also
electrode presets available for different probe types, which can be selected from the dropdown menu
under the *Electrode Preset* label. The presets will automatically select the electrodes for the
probe following the rules described above. 

Calibration Files
^^^^^^^^^^^^^^^^^^^^^

Neuropixels probes require calibration files to be loaded in order to stream data correctly. The
calibration files can be loaded by clicking the :kbd:`...` button next to the respective file. This
will open a file dialog that allows you to select the calibration file for the probe. The calibration
file must be in the format specified by the Neuropixels documentation, and the naming scheme must
match the respective calibration file for the probe. This typically follows the pattern: 
``<probe_number>_<calibration_type>.csv``, where `<probe_number>` is the serial number of the probe,
and `<calibration_type>` is the type of calibration (e.g., `ADC`, `Gain`, etc.).

BNO Configuration
----------------------

Currently there are no settings available for the BNO IMU in the Onix Source plugin. The BNO IMU can
be enabled or disabled by clicking the "Enable/Disable" button in the BNO055 tab. When enabled, the
BNO IMU will stream data to the GUI, and the data can be visualized in the GUI's main control panel.
The BNO IMU data will be streamed as a separate data stream, and can be visualized using the "LFP
Viewer" plugin.

All channels from the BNO IMU will be streamed, and there are no options to select which channels to
stream. The BNO IMU data will be streamed at 100 Hz. Each BNO IMU stream will have the following
channels:

- Euler angles (roll, pitch, yaw)
- Quaternion (x, y, z, w)
- Acceleration (x, y, z)
- Gravity (x, y, z)
- Temperature (Celsius)
- Calibration status (magnetometer, accelerometer, gyroscope, system)
  
  - Values are [0-3], where 0 means not calibrated and 3 means fully calibrated for that data type.
