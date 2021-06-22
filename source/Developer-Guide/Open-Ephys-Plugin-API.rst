.. _openephyspluginAPI:

.. default-domain:: cpp

Open Ephys Plugin API
=====================

The Open Ephys Plugin API consists of all of the classes and methods that can be called by plugins. The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated by the :code:`PLUGIN_API` macro. 

.. note:: This documentation is focused on "Processor" plugins, which inherit from the :code:`GenericProcessor` class. Not all of the methods described below are available to other types of plugins, such as DataThreads, RecordEngines, and FileSources.

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

This guide will cover the main methods that plugins can use to handle continuous, event, and spike data under the three situations listed above.

Building the Signal Chain
#########################

When your plugin is dropped into the signal chain, the GUI will call :code:`GenericProcessor::update()` to update its settings. This will automatically copy all of the settings from the upstream plugin into three "channel info" arrays:

#. :code:`dataChannelArray`
#. :code:`eventChannelArray`
#. :code:`spikeChannelArray`

The GUI will then call :code:`createEventChannels()` and :code:`createSpikeChannels()`, to allow the plugin to optionally append its own :code:`EventChannel` or :code:`SpikeChannel` objects to these arrays. The plugin should override these methods if it generates event or spike data.

Finally, the GUI will call :code:`GenericProcessor::updateSettings()`, which allows the plugin to edit its channel objects. For example, a plugin could change the name of a data channel, re-order channels, or add metadata to a channel object. Note that updating these channel objects is the *only* way that a plugin can communicate its settings to downstream plugins. Any local settings that are not applied to these channel objects will not be visible to other plugins.

There are three types of continuous channels that plugins will encounter:

#. :code:`DATA` - default continuous channel type, typically used to represent a voltage trace recorded from an electrode implanted in the brain. Data is stored in :code:`float` format, in units of **microvolts**.
#. :code:`AUX` - auxiliary data channel, typically used for sensors mounted on a subject's head (such as an accelerometer). Data is stored in :code:`float` format, in arbitrary units.
#. :code:`ADC` - generic analog-to-digital converter channel, typically used for non-head-mounted analog inputs. Data is stored in :code:`float` format, in units of **volts**.

There are also three types of event channels:

#. :code:`TTL` - the most common event channel type, representing a set of digital bits that flip between on and off states
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

When data acquisition is active, the GUI will call each plugin's :code:`GenericProcessor::process()` whenever a new data buffer is available. Within this method, the plugin can interact with the continuous data buffer using the following methods:

.. function:: int getNumSamples(int channelIndex)

    Returns the number of samples available for a given channel. Must be called
    on a per-channel basis, because channels from different subprocessors are
    not guaranteed to have the same number of samples in each buffer.

    :param channelIndex: The index of the continuous data channel to check.
    :return: The number of samples available in the current buffer for the specified channel.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/FilterNode/FilterNode.cpp#L260-L270>`__ method.

.. function:: float * getReadPointer(int channelIndex)

    Returns a pointer to the data for one channel; only use this if the plugin will not overwrite the continuous data buffer.

    :param channelIndex: The index of the continuous data channel to access.
    :return: A pointer to the continuous data.
    :example: See the `PhaseDetector::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/PhaseDetector/PhaseDetector.cpp#L241>`__ method.

.. function:: float * getWritePointer(int channelIndex)

    Returns a pointer to the data for one channel; only use this if the plugin will overwrite the continuous data buffer.

    :param channelIndex: The index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/FilterNode/FilterNode.cpp#L260-L270>`__ method.

.. warning:: To avoid returning invalid data (or a segmentational fault), never try to access more samples than are returned by the :code:`getNumSamples()` method for a particular channel.

Event data
----------------

If your plugin needs to respond to incoming events, it should call the following method at the beginning of the :code:`process()` callback:

.. function:: void checkForEvents(bool respondToSpikes = false)

    Indicates that this plugin needs access to the events within the current buffer.

    :param respondToSpikes: Set to :code:`true` if the plugin needs to process incoming spikes. Defaults to :code:`false`.
    :example: See the `ArduinoOutput::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/ArduinoOutput/ArduinoOutput.cpp#L201-L204>`__ method.

Override the following :code:`GenericProcessor` method to respond to events:

.. function:: void handleEvent(const EventChannel* eventChannel, const MidiMessage& event, int samplePosition)

    Passes the next available incoming event to the plugin.

    :param eventChannel: Pointer to the EventChannel object containing information about the channel that generated this event.
    :param event: MidiMessage object that holds the event data.
    :param samplePosition: The sample within the current continuous buffer at which the event occurred.
    :example: See the `ArduinoOutput::handleEvent() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/ArduinoOutput/ArduinoOutput.cpp#L101-L138>`__ method.

To add an event inside the :code:`process()` loop, call the following method:

.. function:: void addEvent(const EventChannel* eventChannel, const MidiMessage& event, int samplePosition)

    Adds an event on the specified channel, which will be seen by downstream plugins.

    :param eventChannel: Pointer to the EventChannel object containing information about the channel that generated this event.
    :param event: MidiMessage object that holds the event data.
    :param samplePosition: The sample within the current continuous buffer at which the event occurred.
    :example: See the `PhaseDetector::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/PhaseDetector/PhaseDetector.cpp#L247-L254>`__ method.

Spike data
----------------

Assuming :code:`checkForEvents(true)` has already been called, override the following :code:`GenericProcessor` method to respond to spikes:

.. function:: void handleSpike(const SpikeChannel* spikeChannel, const MidiMessage& event, int samplePosition)

   Passes the next available spike to the plugin.

   :param spikeChannel: Pointer to the SpikeChannel object containing information about the channel that generated this event.
   :param event: MidiMessage object that holds the spike data.
   :param samplePosition: The sample within the current continuous buffer at which the spike occurred.
   :example: See the `EventTrigAvg::handleSpike() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/EvntTrigAvg/EvntTrigAvg.cpp#L190-L231>`__ method.

To add a spike inside the :code:`process()` loop, call the following method:

.. function:: void addSpike(const SpikeChannel* spikeChannel, const SpikeEvent* spike, int samplePosition)

   Indicates that this plugin needs access to the events within the current buffer.

   :param spikeChannel: Pointer to the SpikeChannel object containing information about the channel that generated this spike.
   :param event: SpikeEvent object that holds the spike data.
   :param samplePosition: The sample within the current continuous buffer at which the spike occurred.
   :example: See the `SpikeSorter::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/SpikeSorter/SpikeSorter.cpp#L988-L990>`__ method.


Loading/Saving Settings
#######################

The GUI saves the signal chain in the following situations:

#. Whenever a processor is added, moved, or deleted, the signal chain is written to :code:`recoveryConfig.xml`
#. Whenever a recording is started, the signal channel is written to :code:`settings.xml` inside each Record Node directory
#. Whenever the GUI is closed, the signal chain is written to :code:`lastConfig.xml`
#. Whenever the signal chain is cleared, the previous state is stored in memory so this action can be done.

In addition, the settings for individual plugins are stored in memory whenever a plugin is copied.

In order to make sure its settings can be appropriately saved and restored, a plugin must override the following :code:`GenericProcessor` methods:

.. function:: void loadCustomParametersFromXml()

    This method allows a plugin to restore its settings from its :code:`parametersAsXml` member.

    :example: See the `PulsePalOutput::loadCustomParametersFromXml() <https://github.com/open-ephys/plugin-GUI/blob/ea57b8a14f3c3231a0c96ede61d62119b59cc6a5/Plugins/PulsePalOutput/PulsePalOutput.cpp#L463-L510>`__ method.

.. function:: void saveCustomParametersToXml(XmlElement* parentElement)

    This method allows a plugin to save its settings to an `XmlElement <https://docs.juce.com/master/classXmlElement.html>`__ object.

    :param parentElement: The XmlElement containing the plugin's settings (assuming :code:`loadCustomParametersFromXml` has been implemented.
    :example: See the `PulsePalOutput::saveCustomParametersToXml() <https://github.com/open-ephys/plugin-GUI/blob/ea57b8a14f3c3231a0c96ede61d62119b59cc6a5/Plugins/PulsePalOutput/PulsePalOutput.cpp#L436-L461>`__ method.

Plugins can also save and load settings via their editors, by overriding the :code:`GenericEditor::saveCustomParameters()` and :code:`GenericEditor::loadCustomParameters()` methods.

.. note:: Plugins that include a visualizer must use a different set of methods for loading/saving settings from their editors: :code:`VisualizerEditor::saveVisualizerParameters()` and :code:`VisualizerEditor::loadVisualizerParameters()` methods.

Core Services
###############

Plugins also have access to the methods defined in `CoreServices.h <https://github.com/open-ephys/plugin-GUI/blob/master/Source/CoreServices.h>`__. Two of the most commonly used ones are documented below. See the source code for a complete list of available methods. 

.. function:: void sendStatusMessage(String& messageText)

   Sends a status message to the user, which will appear in the Message Center at the bottom of the GUI window.

   :param messageText: The message to be displayed.


.. function:: void updateSignalChain(GenericEditor* editor)

   Indicates that downstream plugins need to be notified about new settings. For example, this should be called when a plugin adds or removes a spike channel, event channel, or data channel; adds metadata to a channel; or changes its "enabled" status.

   :param editor: A pointer to the plugin's editor. 


Other features of the Plugin API
#################################

The Plugin API includes convenient classes for building user interfaces, such as buttons, icons, sliders, and simple line plots. See the :code:`Source/Processors/Editors` and the :code:`Source/Processors/Visualization` directories for more information.

In addition, plugins can access a wide range of digital filters from an `MIT-licensed DSP library <https://github.com/vinniefalco/DSPFilters>`__. See the :code:`Source/Processors/Dsp` directory for a complete list, and check out the `FilterNode <https://github.com/open-ephys/plugin-GUI/tree/master/Plugins/FilterNode>`__ for an example of how these can be used.