.. _openephyspluginAPI:

.. default-domain: cpp

Open Ephys Plugin API
=====================

The Open Ephys Plugin API consists of all of the classes and methods that can be called by plugins. The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated by the :code:`PLUGIN_API` macro. All plugins are sub-classes of the :code:`GenericProcessor` class, and will therefore inherit all of its methods. Additional classes and methods available to the Plugin API will be described below.

Overview
#####################

There are three primary situations in which information is transferred in/out of plugins: 

#. while building the signal chain
#. during acquisition
#. when loading/saving settings.

In each of these situations, there are three primary types of data that can be handled by plugins:

#. continuous data
#. events
#. spikes

This guide will cover the main methods that are used to configure continuous, event, and data channels under the three situations listed above. These instructions are targeted to "filter" or "sink" processor plugins that receive data. If you're creating a DataThread, RecordEngine, or FileSource plugin, the interfaces may differ.

Building the Signal Chain
#########################

When your plugin is dropped into the signal chain, the GUI will immediately call :code:`GenericProcessor::update()` to update its settings. This will automatically copy all of the settings from the upstream plugin into three "channel info" arrays:

#. :code:`dataChannelArray`
#. :code:`eventChannelArray`
#. :code:`spikeChannelArray`

It will then call :code:`createEventChannels()` and :code:`createSpikeChannels()`, to allow the plugin to optionally append its own :code:`EventChannel` or :code:`SpikeChannel` objects to these arrays.

Finally, the GUI will call :code:`GenericProcessor::updateSettings()`, which allows the plugin to edit the available channel objects. For example, a plugin could change the name of a data channel, re-order channels, or add metadata to a channel object. Note that updating these channel objects is the *only* way that a plugin can communicate its settings to downstream plugins. Any local settings that are not applied to these channel objects will not be visible to other plugins.

There are three types of continuous (data) channels that plugins will encounter:

#. :code:`DATA` - default continuous channel type, typically used to represent a voltage trace recorded from an electrode implanted in the brain. Data is stored in :code:`float` format, in units of **microvolts**.
#. :code:`AUX` - auxiliary data channel, typically used for sensors mounted on a subject's head (such as accelerometer)
#. :code:`ADC` - generic analog-to-digital converter channel, typically used for non-head-mounted analog inputs. Data is stored in :code:`float` format, in units of **volts**.

There are also three types of event channels:

#. :code:`TTL` - the most common event channel type, representing a set of digital inputs that flip between on and off states
#. :code:`TEXT` - an event channel that carries text strings
#. :code:`BINARY` - an event channel that transmits user-defined blobs of binary data (not commonly used)

Finally, there are three types of spike channels:

#. :code:`SINGLE` - a spike channel associated with a single continuous channel
#. :code:`STEREOTRODE` - a spike channel associated with two continuous channels
#. :code:`TETRODE` - a spike channel associated with four continuous channels

Data Acquisition
#####################

Continuous data
----------------

When data acquisition is active, the GUI will start calling each plugin's :code:`GenericProcessor::process()` whenever a new data buffer is available. Within this method, the plugin can interact with the continuous data buffer using the following methods:

.. cpp:function:: int getNumSamples(int channel_index)

    Returns the number of samples available for a given channel. Must be called
    on a per-channel basis, because channels from different subprocessors are
    not guaranteed to have the same number of samples in each buffer.

    :param channel_index: The index of the continuous data channel to check.
    :return: The number of samples available in the current buffer for that channel.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

.. function:: float* buffer.getReadPointer(int channel_index)

    Returns a pointer to the data for one channel; only use this if the plugin will not overwrite the continuous data buffer.

    :param channel_index: The index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `PhaseDetector::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/PhaseDetector/PhaseDetector.cpp>`__ method.

.. function:: float* buffer.getWritePointer(int channel_index)

    Returns a pointer to the data for one channel; only use this if the plugin will overwrite the continuous data buffer.

    :param channel_index: The index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

.. warning:: To avoid returning invalid data (or a segmentational fault), never try to access more samples than are returned by the :code:`getNumSamples` method for that particular channel.

Event data
----------------

If your plugin needs to respond to incoming events, it should call the following method at the beginning of the :code:`process()` callback:

.. function:: void checkForEvents(bool respondToSpikes = false)

    Indicates that this plugin needs access to the events within the current buffer.

    :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

Implement the following method to respond to events:

.. function:: void handleEvent(const EventChannel* eventInfo, const MidiMessage& event, int samplePosition = 0)

    Indicates that this plugin needs access to the events within the current buffer.

    :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

To add an event, call the following method:

.. function:: void handleEvent(const EventChannel* eventInfo, const MidiMessage& event, int samplePosition = 0)

    Indicates that this plugin needs access to the events within the current buffer.

    :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

Spike data
----------------

Assuming `checkForEvents(true)` has already been called, implement the following method to respond to spikes:

.. function:: void handleEvent(const EventChannel* eventInfo, const MidiMessage& event, int samplePosition = 0)

   Indicates that this plugin needs access to the events within the current buffer.

   :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
   :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

To add a spike to the data buffer, call the following method:

.. function:: void handleEvent(const EventChannel* eventInfo, const MidiMessage& event, int samplePosition = 0)

      Indicates that this plugin needs access to the events within the current buffer.

      :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
      :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.


Loading/Saving Settings
-----------------------

.. function:: void loadCustomParametersFromXml()

    Returns a pointer to the data for one channel; only use this if the plugin will overwrite the continuous data buffer.

    :param channel_index: The index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

.. function:: void saveCustomParametersToXml(XmlElement* parentElement)

    Returns a pointer to the data for one channel; only use this if the plugin will overwrite the continuous data buffer.

    :param channel_index: The index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/master/Plugins/FilterNode/FilterNode.cpp>`__ method.

Core Services
###############

text goes here.


User Interface Classes
######################

text goes here.

Digital Filters
################

text goes here.

