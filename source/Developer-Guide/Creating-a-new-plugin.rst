.. _creatinganewplugin:
.. role:: raw-html-m2r(raw)
   :format: html

Creating a new plugin
================================

Plugins are the primary means of extending the GUI's functionality. You can create "source" plugins that interface with new types of data acquisition hardware, "filter" plugins that modify incoming data, or "sink" plugins that send data to other applications or output devices. "Record engines" are also plugins that define new formats for reading an writing data.

Some plugins still live inside the main `plugin-GUI <https://github.com/open-ephys/plugin-GUI>`__ repository, but we recommend that all new plugins be hosted separately.

Some examples of externally hosted plugins include:

* `Neuropixels PXI <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ (DataThread Source)
* `Stream Muxer <https://github.com/open-ephys-plugins/StreamMuxer>`__ (Filter Processor)
* `Probe Viewer <https://github.com/open-ephys-plugins/probe-viewer>`__ (Sink Processor)
* `NWB Format <https://github.com/open-ephys-plugins/NWBFormat>`__ (RecordEngine)

This page will show you how to create new plugins and compile them for Linux, Mac, and Windows. These instructions assume you have already compiled the main application. If not, follow the instructions on :ref:`this page <compilingthegui>`.

Creating a new plugin repository
#################################

The first step in creating a new plugin is to create a repository from the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to https://github.com/open-ephys-plugins/OEPlugin 

3. Click the green "Use this template" button.

4. Choose a name for your plugin. It should succinctly capture the plugin's functionality (and doesn't need to include the word "plugin").

5. Click the green "Create repository from template" button.

On your local machine, create an "OEPlugins" directory within the same directory that contains your :code:`plugin-GUI` repository: Then, using the command line or the `GitHub Desktop <https://desktop.github.com/`__ app, clone your the plugin repository into this new folder. Your directory structure should look something like this:

.. code-block:: 

   code_directory/
      plugin-GUI/
      OEPlugins/
         NewPlugin/
            Source/
            Build/
            CMakeLists.txt
            CMAKE_README.txt
            README.md


Editing :code:`OpenEphysLib.cpp`
#####################################

Inside the "Source" folder of the plugin repository you just cloned, you'll find the :code:`OpenEphysLib.cpp` file that contains critical information about your plugin. Open it in your preferred text editor and make the following changes, depending on the type of plugin you're creating:

**Processor** Plugins
----------------------

Most plugins will be "processors," meaning they implement the :code:`process()` method of `GenericProcessor.cpp <https://github.com/open-ephys/plugin-GUI/blob/master/Source/Processors/GenericProcessor/GenericProcessor.cpp>`__. This method is called repeatedly during the GUI's acquisition loop, so each plugin has a chance to respond to incoming data (or generate its own). Processor plugins should uncomment and edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++
   
   // specifies that we are creating a Processor plugin
   info->type = PluginType::PLUGIN_TYPE_PROCESSOR;

   // edit to change how your plugin's name is displayed in the GUI
   info->processor.name = "Plugin Name"; 

   // Select one of the following: FilterProcessor, SourceProcessor, SinkProcessor or UtilityProcessor
   info->processor.type = ProcessorType::FilterProcessor; 

   // Replace "ProcessorPluginSpace::ProcessorPlugin" with the namespace and class name of your plugin
   info->processor.creator = &(Plugin::createProcessor<ProcessorPluginSpace::ProcessorPlugin>);

**DataThread** Plugins
------------------------

Source plugins that acquire data in a separate thread (e.g., because the acquisition hardware is not sychronized with the computer's clock) should be implemented as a DataThread plugin. Instead of calling the :code:`process()` method directly, DataThread plugins add data to a buffer as it becomes available, which will be automatically copied into the GUI's signal chain. DataThread plugins should uncomment and edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++
   
   // specifies that we are creating a DataThread plugin
   info->type = PluginType::PLUGIN_TYPE_DATA_THREAD;

   // edit to change how your plugin's name is displayed in the GUI
   info->dataThread.name = "Plugin Name"; 

   // Replace "DataThreadClassName" with the class name of your plugin
   info->dataThread.creator = &createDataThread<DataThreadClassName>;

**RecordEngine** and **FileSource** Plugins
--------------------------------------------

See :ref:`addinganewdataformat`.

Adding source code
#################################

Add the plugin's source files to the "Source" directory, or use the included files as a starting point.

If you're using the template files, choose the header and cpp files corresponding to the type of plugin you're creating (Processor, DataThread, RecordEngine, or FileSource), and delete the rest. you'll have to find and replace the default class name with the name of your plugin's class.

By default, CMake will add any files with **.h** or **.cpp** extensions that live in the "Source" directory. If you have files with alternate extensions, you'll have to edit the following line of **CMakeLists.txt**:

.. code-block::

   file(GLOB_RECURSE SRC_FILES LIST_DIRECTORIES false "${SOURCE_PATH}/*.cpp" "${SOURCE_PATH}/*.h")


Including external libraries
################################

If your plugin depends on external libraries, it is necessary to manually edit the **CMakeLists.txt** file. The relevant lines are commented out at the end of this file.


Compiling your plugin
#########################

Follow the instructions on :ref:`compilingplugins` to build your new plugin.



.. |br| raw:: html

  <br/>
