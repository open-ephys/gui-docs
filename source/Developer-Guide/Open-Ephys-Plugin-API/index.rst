.. _openephyspluginAPI:

.. default-domain:: cpp

Open Ephys Plugin API
=====================

Overview
#########

The Open Ephys Plugin API consists of all of the classes and methods that are accessible by plugins. In order to extend the functionality of the GUI, one only needs to understand the interfaces that compose the API for a particular plugin type. And because plugins are compiled independently from the core application, it is straightforward to share this new functionality with other users.

The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated in the source code by the :code:`PLUGIN_API` macro.

Detailed information about the five different types of plugins that can be created is available on the following pages:

.. toctree::
    :maxdepth: 1

    Processor-Plugins
    Visualizer-Plugins
    Data-Threads
    Record-Engines 
    File-Sources

In addition, some general features of the plugin API are covered below:


Logging
#########

When plugins need to log information, they should ideally use the GUI's built-in logger class, defined in :code:`Utils.h`. This is preferred to calling :code:`std::cout` or `printf`, because the logger class will (optionally) print output to the console as well as to a log file that is created each time the GUI is run.

The two primary methods are:

- :code:`LOGC` ("log console"): prints info to the console and also saves it to a log file
- :code:`LOGD` ("log debug"): prints info to the console in debug mode, and saves it to a log file in release mode

These are used by calling, e.g.:

.. code-block:: c++

    LOGC(arg1, arg2, arg3, ...);

where the arguments are strings, integers, floats, etc. -- any values that can be converted to strings by `std::cout`. There is no limit to the number of arguments that can be used.

In addition, there are several other methods that are reserved for specific cases:

- :code:`LOGA` ("log action"): writes a line in the log file whenever the user performs an action (e.g. pressing a button, updating a label, etc.).
- :code:`LOGB` ("log buffer"): logs messages inside the :code:`process()` callback (never do this in release mode).
- :code:`LOGE` ("log error"): prints the console output via :code:`std::cerr` instead of :code:`std::cout`.
- :code:`LOGF` ("log file"): prints to the log file only.
- :code:`LOGG` ("log graph"): messages related to constructing the GUI's processor graph.

Core Services
##############

All types of plugins can access the :code:`CoreServices` class, which includes the following general-purpose methods:

.. function:: String CoreServices::getGUIVersion()

    Returns the version of the host application as a string. This can be useful if certain aspects of a plugin's functionality requires a minimum version of the GUI.


.. function:: File CoreServices::getDefaultUserSaveDirectory()

    Returns a Juce :code:`File` object that specifies the location where files are saved by default (typically in the "Documents" directory). This is useful if a plugin needs to save custom data files.


.. function:: File CoreServices::getSavedStateDirectory()

    Returns a Juce :code:`File` object that specifies the location where the GUI's configuration files are stored. This is useful if a plugin needs to save a custom settings file (although it's recommended to use the GUI's built-in XML settings file whenever possible).


.. function:: void CoreServices::saveRecoveryConfig()

    Triggers the GUI to save the recovery configuration file (:code:`recoveryConfig.xml`). This file is re-generated automatically whenever the signal chain is updated, but a plugin may wish to re-generate it when its internal parameters have changed, so these can be re-loaded in the event of a crash.


.. function:: void CoreServices::updateSignalChain(GenericEditor* editor)

    Updates all processors downstream of the specified :code:`GenericEditor`. This should be used whenever a plugin adds or remove continuous, spike, or event channels, or changes the properties of these channels (such as their name or other metadata).

    :param editor: A pointer to the editor of the processor that has been modified.


.. function:: void CoreServices::highlightEditor(GenericEditor* editor)

    Highlights a plugin's editor in the Editor Viewport, without updating the signal chain. This will also make the editor visible if it's not currently seen by the user.

    :param editor: The editor to highlight.


.. function:: void CoreServices::loadSignalChain(String path)

    Loads a signal chain from the specified absolute path. Note that this will delete all existing plugins from the signal chain.

    :param path: The path to a valid XML Open Ephys settings file.


.. function:: bool CoreServices::getAcquisitionStatus()

    Returns true if acquisition is active. If a plugin needs to check this frequently, it's recommended to store a local variable that's updated inside the :code:`GenericProcessor::startAcquisition()` and :code:`GenericProcessor::stopAcquisition()` methods.


.. function:: void CoreServices::setAcquisitionStatus(bool enable)

    Turns acquisition on and off. This could be used to start and stop acquisition when a particular external trigger is received.

    :param enable: Set to :code:`true` to start acquisition, and :code:`false` to stop acquisition.


.. function:: bool CoreServices::getRecordingStatus()

    Returns true if recording is active. If a plugin needs to check this frequently, it's recommended to store a local variable that's updated inside the :code:`GenericProcessor::startRecording()` and :code:`GenericProcessor::stopRecording()` methods.


.. function:: void CoreServices::setRecordingStatus(bool enable)

    Turns recording on and off. This could be used to start and stop recording when a particular external trigger is received.

    :param enable: Set to :code:`true` to start recording, and :code:`false` to stop recording.


.. function:: void CoreServices::sendStatusMessage(const String& text)

    Display a message to the user in the Message Center at the bottom of the screen. These messages will not be saved by the Record Node; to write messages that are saved, call :code:`GenericProcessor::broadcastMessage()` while recording is active.

    :param text: The message to display.


.. function:: int64 CoreServices::getSoftwareTimestamp()

    Returns the current software timestamp (milliseconds since midnight Jan 1st 1970 UTC).


.. function:: float CoreServices::getSoftwareSampleRate();

    Returns the ticker frequency of the software timestamp clock (1000 Hz).


.. function:: File CoreServices::getRecordingParentDirectory()

    Returns the default recording directory.


.. function:: void CoreServices::setRecordingParentDirectory(String dir)

    Sets new default recording directory. This will only affect new Record Nodes, and will not be applied to Record Nodes that are already in the signal chain. To set the recording directory for existing Record Nodes, use :code:`CoreServices::RecordNode::setRecordingDirectory()` (see below).

    :param dir: The absolute path of the new recording directory.


.. function:: void CoreServices::RecordNode::setRecordingDirectory(String dir, int nodeId, bool applyToAll = false)
    
    Sets new recording directory for one or all existing Record Nodes.

    :param dir: The absolute path of the new recording directory.
    :param nodeId: The ID of the Record Node (if only one is being updated).
    :param applyToAll: Set to :code:`true` to apply the new directory to all Record Nodes.


.. function:: Array<int> CoreServices::getAvailableRecordNodeIds()

    Returns an array of IDs for Record Nodes currently in the signal chain.


.. function:: void CoreServices::setRecordingDirectoryBasename(String dir)

    Sets new basename for the recording directory (does not affect prepend/append text)


.. function:: String CoreServices::getRecordingDirectoryName()

    Returns the full name of the current recording directory (empty string if none has started)


.. function:: void CoreServices::createNewRecordingDirectory()

    Creates new directory the next time recording is started.


.. function:: String CoreServices::getRecordingDirectoryPrependText()

    Set the text to be prepended to the name of newly created recording directories.


.. function:: String getRecordingDirectoryAppendText()

    Set the text to be appended to the name of newly reated recording directories.


.. function:: void setRecordingDirectoryPrependText(String text)

    Set the text to be prepended to the name of newly created recording directories.


.. function:: void setRecordingDirectoryAppendText(String text)

    Set the text to be appended to the name of newly created recording directories.

    :param text: The 


.. function:: bool allRecordNodesAreSynchronized()

    Returns :code:`true` if all Record Nodes are in a "synchronized" state. A Record Node is synchronized if it only has one data stream as input, or if all of its incoming streams share a hardware sync line that has received at least two events.

