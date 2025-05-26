.. _processorplugins:

.. default-domain:: cpp

=====================
Processor Plugins
=====================

.. csv-table:: Processor plugins are the key elements of the GUI's signal chain; they respond to and update continuous data, spikes, and events inside their :code:`process()` method.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::PROCESSOR`"
   "*Base Classes*", ":code:`GenericProcessor`, :code:`GenericEditor`"
   "*Template*", "https://github.com/open-ephys-plugins/processor-plugin-template"

Overview
#####################

If you're developing a new plugin for the GUI, it will most likely be a **Processor Plugin**. The core functionality of these plugins is defined in their :code:`process()` method, which determines how they generate, modify, or respond to data passing through the GUI's signal chain.

There are three type of Processor Plugins:

#. **Sources** generate data, and must be placed at the beginning of a signal chain. Note that if a source is communicating with external hardware that is not synchronized with the GUI's signal chain callbacks, a :ref:`datathreads` plugin should be used instead.
#. **Filters** modify data, either by changing the samples contained in the continuous channel data buffer or adding TTL events or spikes to the event buffer. Filters must receive data from at least one Source.
#. **Sinks** send data outside the signal chain, without modifying it in any way. Sinks are typically used for visualizing data or sending triggers to external hardware.

Processor Plugins must also implement an "Editor" derived from the :code:`GenericEditor` class which contains various UI elements for changing the plugin's parameters.

In addition, Processor Plugins can be "Visualizers" if they include a canvas for displaying data and/or parameter interfaces. See the :ref:`visualizerplugins` page for more information about the API methods related to visualization.


Key methods
################################

Constructors
=============

GenericProcessor constructor
-----------------------------

The plugin class constructor should look like this:

.. code-block:: c++

    ProcessorPlugin::ProcessorPlugin() 
        : GenericProcessor ("Plugin Name")
    {

    }

The :code:`GenericProcessor` constructor includes one parameter that specifies how the plugin's name will appear in the GUI's Processor List. The plugin's constructor can be used to set the initialize state of key variables (but not that that any :code:`Parameter` objects should be initialized in the :code:`registerParameters()` method).

GenericEditor constructor
-----------------------------

The constructor for the plugin's editor class must also be defined as follows:

.. code-block:: c++

    ProcessorPluginEditor::ProcessorPluginEditor(GenericProcessor* parentNode) 
        : GenericEditor(parentNode)
    {
        desiredWidth = 150;

        // add parameter editors here
    }

You can change the value of :code:`desiredWidth` to specify how wide (in pixels) the plugin's editor should be. This value can be changed later on if the editor needs to expand or contract. The editor constructor must define any :code:`ParameterEditor` widgets that will be used. The underlying parameters must have already been defined in the plugin's constructor.

Next, you'll need to make sure the plugin's :code:`createEditor()` method is correct:

.. code-block:: c++

    AudioProcessorEditor* ProcessorPlugin::createEditor()
    {
        editor = std::make_unique<ProcessorPluginEditor>(this);
        return editor.get();
    }

.. note:: ":code:`ProcessorPluginEditor`" should be changed to match the name of your plugin's editor class.

Updating plugin settings
===========================

Whenever the signal chain is modified, the GUI will call :code:`updateSettings()` on all plugins downstream of the modification, to allow them to respond to changes and inform downstream plugins about their current state:

.. function:: void updateSettings()

    Processor plugins should override this method in order to respond to configuration information about upstream plugins, and ensure their own configuration information is sent to downstream plugins.

The following internal variables are used to pass information between plugins prior to the start of acquisition:

* :code:`dataStreams` - A Juce :code:`OwnedArray` that stores pointers to the available :code:`DataStream` objects that will be processed by this plugin. Each :code:`DataStream` is assigned a :code:`uint16` identifier that is guaranteed to be unique within a given run of the GUI, but does not persist between runs.

* :code:`continuousChannels` - A Juce :code:`OwnedArray` that stores pointers to the available :code:`ContinuousChannel` objects that will be processed by this plugin. Each :code:`ContinuousChannel` must be associated with a :code:`DataStream` containing all of the other channels that are sampled synchronously.

* :code:`eventChannels` - A Juce :code:`OwnedArray` that stores pointers to the available :code:`EventChannel` objects that will be processed by this plugin. Similar to :code:`ContinuousChannel` objects, each :code:`EventChannel` must be associated with one and only one :code:`DataStream`. Note that each :code:`EventChannel` object can track state changes across many TTL "lines." 

* :code:`spikeChannels` - A Juce :code:`OwnedArray` that stores pointers to the available :code:`SpikeChannel` objects that will be processed by this plugin. Each :code:`SpikeChannel` must be associated with one and only one :code:`DataStream`, and will also contain pointers to the :code:`ContinuousChannel` objects that represent the continuous channels from which spikes are samples. A :code:`SpikeChannel` will generate spikes that are associated with one "Electrode" (e.g. single electrode, stereotrode, or tetrode).

.. important:: :code:`DataStream`, :code:`ContinuousChannel`, :code:`EventChannel`, and :code:`SpikeChannel` objects do *not* persist between calls to :code:`updateSettings()`. Do not store pointers to these objects outside the standard locations, as the underlying objects will be deleted the next time the signal chain is modified.

If a plugin will generate continuous, event, or spike data, it must create new channel info objects and add them to the appropriate :code:`DataStream` inside the :code:`updateSettings()` method.

Plugins that only generate events can call the following method inside :code:`updateSettings()` to automatically add an event channel to the first incoming :code:`DataStream`:

.. function:: void addTTLChannel(String name)

    Adds a TTL event channel to the first incoming data stream, so downstream plugins can respond to events generated by this plugin. This method should only be called once, inside the :code:`updateSettings()` method. Note that each "channel" can contain state information for up to 256 TTL lines.  

    :param name: The name of this channel. Channel names must be unique for each data stream within a particular plugin.


Starting/stopping acquisition
===============================

If a plugin needs to block acquisition from starting, it can override the following method:

.. function:: bool isReady()

    Informs a plugin that acquisition is about to begin. The plugin should return :code:`true` to confirm that acquisition can be safely started.

    :return: :code:`true` if acquisition can be started, :code:`false` otherwise.

Ideally this should be accompanied by a status message stating the reason why acquisition could not begin.

If a plugin needs to update its internal state when acquisition starts or stops, it should override the following method:

.. function:: bool startAcquisition()

    Informs a plugin that acquisition is about to begin. 

.. function:: bool stopAcquisition()

    Informs a plugin that acquisition has stopped.


Processing data
===================

The :code:`process()` method is where a Processor Plugin's core functionality is defined:

.. function:: void process(AudioBuffer<float>& buffer)

    Allows a plugin to modify a buffer of continuous channels, and add events or spikes that will be received by downstream plugins. This method is called repeatedly whenever acquisition is active.

    :param buffer: A Juce :code:`AudioBuffer` containing data samples for all continuous channels processed by this plugin. The ordering of channels in this buffer matches the ordering of channels in the :code:`continuousChannels` array. Since this buffer contains samples for all incoming data streams, be sure to call :code:`getGlobalIndex()` for each continuous channel to find its position in this buffer, which may be different from its position within its data stream. In general, this buffer stores data in units of microvolts, but there are some cases (such as ADC channels) where the data is stored in volts.

Continuous data
-----------------

Before performing any read/write operations on a continuous channel, it is necessary to query the number of valid samples that are available in a given callback:

.. function:: int getNumSamplesInBlock(uint16 streamId)

    Returns the number of samples available for a stream. Must be called
    on a per-stream basis, because different streams are
    not guaranteed to have the same number of samples in each buffer.

    :param streamId: The ID of the data stream to check.
    :return: The number of samples available in the current buffer for the specified stream.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/FilterNode/FilterNode.cpp#L260-L270>`__ method.

A plugin should never request samples that are above the index returned by this method. This method can sometimes return 0, if there are no samples available for a given data stream.

In order to read or write data from the continuous buffer, the following methods should be used:

.. function:: float * getReadPointer(int globalChannelIndex)

    Returns a pointer to the data for one channel; only use this if the plugin will not overwrite the continuous data buffer.

    :param globalChannelIndex: The global index of the continuous data channel
    :return: A pointer to the continuous data.
    :example: See the `PhaseDetector::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/PhaseDetector/PhaseDetector.cpp#L241>`__ method.

.. function:: float * getWritePointer(int globalChannelIndex)

    Returns a pointer to the data for one channel; only use this if the plugin will overwrite the continuous data buffer.

    :param globalChannelIndex: The global index of the continuous data channel to modify.
    :return: A pointer to the continuous data.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/FilterNode/FilterNode.cpp#L260-L270>`__ method.

Spike and event data
---------------------

To notify the GUI that the plugin needs to respond to incoming events and spikes within the :code:`process()` method, the following method must be called:

.. function:: int checkForEvents(bool respondToSpikes)

    Indicates that the plugin wants to respond to incoming events and/or spikes.

    :param respondToSpikes: Set to :code:`true` if the plugin needs to receive spikes as well.
    :return: 0 if there are events available in this buffer, -1 otherwise.

The plugin should override the following methods to actually deal with incoming events and spikes:

.. function:: void handleTTLEvent(TTLEventPtr event)

    Passes the next available incoming event to the plugin.

    :param event: Pointer to a :code:`TTLEvent` object containing information about this event. This includes the event channel that generated it, the ID of the data stream it is associated with, the line on which the event occurred, and the sample number of the event (relative to the start of acquisition).
    :example: See the `ArduinoOutput::handleEvent() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/ArduinoOutput/ArduinoOutput.cpp#L101-L138>`__ method.

.. function:: void handleSpike(SpikePtr event)

    Passes the next available incoming spike to the plugin.

    :param event: Pointer to a :code:`Spike` object containing information about this spike. This include the spike channel that generated it, the ID of the data stream it is associated with, the sample number of the event (relative to the start of acquisition), and the full spike waveform (in units of microvolts).
    :example: See the `SpikeDisplayNode::handleSpike() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/ArduinoOutput/ArduinoOutput.cpp#L101-L138>`__ method.

Assuming that :code:`addTTLChannel()` was called inside the :code:`process()` method (see above), a plugin can add events using the following method:

.. function:: void flipTTLState(int sampleIndex, int lineIndex)

    Adds an event indicating a state change on a particular TTL line.

    :param sampleIndex: The sample index (relative to the start of the current block of the first incoming stream) at which this state change occurred.
    :param lineIndex: The TTL line on which the state change occurred (0-255).

.. function:: void setTTLState(int sampleIndex, int lineIndex, bool state)

    Adds an event with a specified state value (ON or OFF). Note that consecutive "ON" or "OFF" events are valid within software, even if they would be impossible to generate using actual hardware.

    :param sampleIndex: The sample index (relative to the start of the current block of the first incoming stream) at which this state change occurred.
    :param lineIndex: The TTL line on which the state change occurred (0-255).
    :param state: The state of the TTL line (ON = :code:`true`, OFF = :code:`false`).

Sending and receiving messages
================================

While acquisition is not active, plugins can respond to **configuration messages** and send **status messages**:

.. function:: void handleConfigMessage(String message)

    Allows a plugin to respond to a configuration message (usually received via the :code:`OpenEphysHTTPServer`). This makes it possible to configure a plugin's settings remotely. Config messages are ignored if acquisition is active.

     :param message: The content of the configuration message. There are no restrictions on how this string is formatted; each plugin is responsible for parsing this message in the appropriate way.

.. function:: void CoreServices::sendStatusMessage(String message)

    Displays a message to the user in the GUI's Message Center.

     :param message: The message to display.

While acquisition is active, plugins can respond to and send **broadcast messages**:

.. function:: void handleBroadcastMessage(String message)

    Allows a plugin to respond to an event that carries a text value, which is broadcast throughout the signal chain during acquisition. These messages can be used to pass information backwards through the signal chain, e.g. to trigger an output based on events that are generated downstream.

    :param message: The content of the broadcast message. There are no restrictions on how this string is formatted; each plugin is responsible for parsing this message in the appropriate way.


.. function:: void broadcastMessage(String message)

    Can be called during the :code:`process()` method to send a message to all other plugins in the signal chain. These messages will be automatically saved by any Record Nodes in the signal chain.

    :param message: The content of the broadcast message. There are no restrictions on how this string is formatted; each plugin is responsible for generating messages that can be parsed by other plugins.


Plugin parameters
#######################

The GUI's built-in :code:`Parameter` class provides an easy way to manage the parameters for your plugin. Using this class provides the following advantages:

* Parameter names, default values, and ranges are defined with one line of code
* Parameter editors can be auto-generated with a second line of code
* Parameters values can be safely updated while acquisition is active
* The GUI will automatically track parameters across different data streams
* Parameter values will be automatically saved and loaded

Defining parameters
====================

All :code:`Parameter` objects *must* be defined inside your plugin's :code:`registerParameters` method, using the constructor methods for different types of parameters, e.g.:

.. function:: void addIntParameter(Parameter::ParameterScope scope, const String& name, const String& description, int defaultValue, int minValue, int maxValue)

    Adds a parameter that can take integer values.

    :param scope: Use :code:`Parameter::GLOBAL_SCOPE` if the parameter will hold a single value each plugin or :code:`Parameter::STREAM_SCOPE` if the parameter can be changed independently for each incoming data stream.
    :param name: Name of the parameter (cannot have spaces)
    :param description: A one-line description of this parameter
    :param defaultValue: The default value for this parameter
    :param minValue: The minimum value this parameter can take
    :param maxValue: The maximum value for this parameter

See `GenericProcessor.h <https://github.com/open-ephys/plugin-GUI>`__ for a complete list of :code:`Parameter` class constructors.

Accessing parameters
====================

For global parameters, use the following method:

.. function:: Parameter* getParameter(String name)

    Returns a pointer to a global parameter.

    :param name: The name of the parameter.

The easiest way to access stream-specific parameters is through the overloaded bracket operator:

.. code-block:: c++

    (*stream)["parameter_name"]->getValue();

In both cases, requesting a parameter name that doesn't exist will result in a segfault.

Creating parameter editors
========================================

Parameters can be modified using editors that are auto-generated by the GUI. These must be initialized in the class constructor for the plugin's editor, e.g.:

.. function:: void addTextBoxParameterEditor (const String& name, int xPos, int yPos)

    Adds a text box that can be used to modify the values of a :code:`Parameter` object.

    :param name: The name of the parameter. This *must* match the name of a parameter that has been created inside the plugin's constructor.
    :param xpos: The horizontal position (in pixels) of this parameter editor within the plugin's editor (left edge = 0)
    :param ypos: The vertical position (in pixels) of this parameter editor within the plugin's editor (top edge = 0)

See `GenericEditor.h <https://github.com/open-ephys/plugin-GUI>`__ for a complete list of Parameter constructors.

Responding to parameter value changes
========================================

Your plugin can implement a custom response to parameter changes. For example, if the filter high cut changes and a filter needs to be updated. To do this, override this virtual method in your plugin.

.. function:: void parameterValueChanged(Parameter* param) override

    Called whenever a parameter value changes.

    :param param: A pointer to the parameter object that was updated.



Saving and loading custom settings
#####################################

The GUI saves the signal chain in the following situations:

#. Whenever a processor is added, moved, or deleted, the signal chain is written to :code:`recoveryConfig.xml`
#. Whenever a recording is started, the signal chain is written to :code:`settings.xml` inside each Record Node directory
#. Whenever the GUI is closed, the signal chain is written to :code:`lastConfig.xml`
#. Whenever the signal chain is cleared, the previous state is stored in memory so this action can be undone.

In addition, the settings for individual plugins are stored in memory whenever a plugin is copied.

If the plugin uses any settings that are not using the built-in :code:`Parameter` class, it needs to implement the following functions to ensure they are saved and loaded properly:

.. function:: void saveCustomParametersToXml(XmlElement* xml)

    Used to save custom parameters.

    :param xml: Pointer to an XmlElement that will store these parameters.

To add a parameter to the :code:`XmlElement`, use the following code:

.. code-block:: c++

    xml->setAttribute("parameterName", value);

The name string cannot have any spaces, and the value can be a boolean, integer, or string.

.. function:: void loadCustomParametersFromXml(XmlElement* xml)

    Used to load custom parameters.

    :param xml: Pointer to an XmlElement that was saved previously.

To read out the parameters, you can use the following methods:

.. code-block:: c++

    int parameter1Value = xml->getIntAttribute("parameter1Name", 0);
    bool parameter2Value = xml->getBoolAttribute("parameter2Name", false);
    String parameter2Value = xml->getStringAttribute("parameter3Name", "default");

Be sure to supply a default value (the second argument), in case the parameter doesn't exist in the config file being loaded.

Plugins can also save and load settings via their editors, by overriding the :code:`GenericEditor::saveCustomParametersToXml()` and :code:`GenericEditor::loadCustomParametersFromXml()` methods.

