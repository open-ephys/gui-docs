.. _processorplugins:

.. default-domain:: cpp

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

#. **Sources** generate data, and must be placed at the beginning of a signal chain.
#. **Filters** modify data, and must receive data from at least one Source
#. **Sinks** send data outside the signal chain, without modifying it in any way

Processor Plugins must also implement an "Editor" class derived from :code:`GenericEditor` which contains various UI elements for changing the plugin's parameters.

In addition, Processor Plugins can be "Visualizers" if they include a canvas for displaying data and/or parameter interfaces. See the :ref:`visualizerplugins` page for more information about the API methods related to visualization.


Constructors
################################

The plugin class constructor should look like this:

.. code-block:: c++

    ProcessorPlugin::ProcessorPlugin() 
        : GenericProcessor ("Plugin Name")
    {

        // add plugin parameters here
    }

The :code:`GenericProcessor` initializer includes one parameter that specifies how the plugin's name will appear in the GUI's Processor List.

The constructor for the plugin's editor class must also be defined as follows:

.. code-block:: c++

    ProcessorPluginEditor::ProcessorPluginEditor(GenericProcessor* parentNode) 
        : GenericEditor(parentNode)
    {
        desiredWidth = 150;

        // add parameter editors here
    }

You can change the value of :code:`desiredWidth` to specify how wide (in pixels) the plugin's editor should be. This value can be changed later on if the editor needs to expand or contract.

Next, you'll need to make sure the plugin's :code:`createEditor()` method is correct:

.. code-block:: c++

    AudioProcessorEditor* ProcessorPlugin::createEditor()
    {
        editor = std::make_unique<ProcessorPluginEditor>(this);
        return editor.get();
    }

":code:`ProcessorPluginEditor`" should be changed to match the name of your plugin's editor class.

Plugin parameters
#######################

The GUI's built-in :code:`Parameter` class provides an easy way to manage the parameters for your plugin. Using this class provides the following advantages:

* Parameter names, default values, and ranges are defined with one line of code
* Parameter editors can be auto-generated with a second line of code
* Parameters values can be safely updated while acquisition is active
* The GUI will automatically track parameters across different data streams
* Parameter values will be automatically saved and loaded

Defining parameters
---------------------

All parameters *must* be defined inside your plugin's class constructor, using the following methods for different types of parameters:

.. function:: void addBooleanParameter(Parameter::ParameterScope scope, const String& name, const String& description, bool defaultValue)

    Adds a parameter with two states (true or false).

    :param scope: Use :code:`Parameter::GLOBAL_SCOPE` if the parameter will hold a single value each plugin or :code:`Parameter::STREAM_SCOPE` if the parameter can be changed independently for each incoming data stream.
    :param name: Name of the parameter (cannot have spaces)
    :param description: A one-line description of this parameter
    :param defaultValue: The default value for this parameter (true or false)

.. function:: void addIntParameter(Parameter::ParameterScope scope, const String& name, const String& description, int defaultValue, int minValue, int maxValue)

    Adds a parameter that can take integer values.

    :param scope: Use :code:`Parameter::GLOBAL_SCOPE` if the parameter will hold a single value each plugin or :code:`Parameter::STREAM_SCOPE` if the parameter can be changed independently for each incoming data stream.
    :param name: Name of the parameter (cannot have spaces)
    :param description: A one-line description of this parameter
    :param defaultValue: The default value for this parameter
    :param minValue: The minimum value this parameter can take
    :param maxValue: The maximum value for this parameter

Accessing parameters
---------------------

For global parameters, use the following method:

.. function:: Parameter* getParameter(String name)

    Returns a pointer to a global parameter.

    :param name: The name of the parameter.

The easiest way to access stream-specific parameters is through the overloaded bracket operator:

.. code-block:: c++

    (*stream)["parameter_name"]->getValue();

In both cases, requesting a parameter that doesn't exist will result in a segfault.

Creating parameter editors
----------------------------

Text goes here.

Responding to parameter value changes
---------------------------------------

Your plugin can implement a custom response to parameter changes. For example, if the filter high cut changes and a filter needs to be updated. To do this, override this virtual method in your plugin.

.. function:: void parameterValueChanged(Parameter*) override

    Called whenever a parameter value changes.

    :param Parameter*: A pointer to the parameter object that was updated.


:code:`updateSettings()` method
##################################

This method is called whenever the upstream signal chain is updated. Plugins can use this method to add spike or event channels, or respond to information coming from upstream.

The :code:`dataStreams` member holds the data streams that are inherit from upstream. If it's a source plugin, this array will need to be populated. Should not add a data stream unless it's a source!

Adding an event channel
------------------------

.. function:: void addTTLChannel(String name)

    Adds a simple TTL event channel.

    :param name: Name of the event channel that will be created.

Configuration messages
################################

- handleConfigMessage()

Starting / stopping acquistion
################################

Text goes here.

The :code:`process()` method
################################

This is the most important method. Must be present in every processor plugin!

Continuous data
----------------

1. Loop through streams
2. Loop through channels 

.. function:: int getNumSamplesInBlock(uint16 streamId)

    Returns the number of samples available for a stream. Must be called
    on a per-stream basis, because different streams are
    not guaranteed to have the same number of samples in each buffer.

    :param streamId: The ID of the data stream to check.
    :return: The number of samples available in the current buffer for the specified stream.
    :example: See the `FilterNode::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/FilterNode/FilterNode.cpp#L260-L270>`__ method.

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


TTL events
---------------
.. function:: int checkForEvents(bool respondToSpikes)

    Indicates that the plugin wants to respond to incoming events and/or spikes.

    :param respondToSpikes: 

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

Convenient methods:

- flipTTLState()
- setTTLState()

Spikes 
---------------

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
   :param spike: SpikeEvent object that holds the spike data.
   :param samplePosition: The sample within the current continuous buffer at which the spike occurred.
   :example: See the `SpikeSorter::process() <https://github.com/open-ephys/plugin-GUI/blob/ebf64f5fc89dee3cb452eb92f9fb63e04d8a68d0/Plugins/SpikeSorter/SpikeSorter.cpp#L988-L990>`__ method.

Broadcast messages
---------------------
- handleBroadcastMessage()

Saving and loading custom parameters
#####################################

The GUI saves the signal chain in the following situations:

#. Whenever a processor is added, moved, or deleted, the signal chain is written to :code:`recoveryConfig.xml`
#. Whenever a recording is started, the signal channel is written to :code:`settings.xml` inside each Record Node directory
#. Whenever the GUI is closed, the signal chain is written to :code:`lastConfig.xml`
#. Whenever the signal chain is cleared, the previous state is stored in memory so this action can be done.

In addition, the settings for individual plugins are stored in memory whenever a plugin is copied.

If the plugin uses any parameters that are not use the built-in :code:`Parameter` class, it needs to implement the following functions to ensure they are saved and loaded properly:

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