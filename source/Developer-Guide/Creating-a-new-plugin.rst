.. _creatinganewplugin:
.. role:: raw-html-m2r(raw)
   :format: html

Creating a new plugin
================================

Plugins are the primary means of extending the GUI's functionality. You can create :ref:`datathreads` that interface with new types of data acquisition hardware, :ref:`processorplugins` that modify incoming data, :ref:`visualizerplugins` that create real-time displays, :ref:`recordengines` that define new output formats, and :ref:`filesources` that can read in pre-recorded data.

Some plugins still live inside the main `plugin-GUI <https://github.com/open-ephys/plugin-GUI>`__ repository, but we recommend that all new plugins be hosted separately.

Some examples of externally hosted plugins include:

* `Neuropixels PXI <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ (**DataThread**)
* `Multi-Band Integrator <https://github.com/open-ephys-plugins/multi-band-integrator>`__ (**Processor Plugin**)
* `Probe Viewer <https://github.com/open-ephys-plugins/probe-viewer>`__ (**Visualizer Plugin**)
* `NWB Format <https://github.com/open-ephys-plugins/nwb-format>`__ (**Record Engine** & **File Source**)

This page will show you how to create new plugins and compile them for Linux, Mac, and Windows. These instructions assume you have already compiled the main application. If not, follow the instructions on :ref:`this page <compilingthegui>`.

For more information about Plugin API commands used by each plugin type, see the :ref:`openephyspluginapi` documentation.

Creating a new plugin repository
#################################

The first step in creating a new plugin is to create a repository from the appropriate plugin template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to the `Processor Plugin <https://github.com/open-ephys-plugins/processor-plugin-template>`__, `Visualizer Plugin <https://github.com/open-ephys-plugins/visualizer-plugin-template>`__, `Data Thread <https://github.com/open-ephys-plugins/data-thread-template>`__, `Record Engine <https://github.com/open-ephys-plugins/record-engine-template>`__, or `File Source <https://github.com/open-ephys-plugins/file-source-template>`__ template repository.

3. Click the green "Use this template" button.

4. Choose a name for your plugin. It should succinctly capture the plugin's functionality (and doesn't need to include the word "plugin").

5. Click the green "Create repository from template" button.

On your local machine, create an "OEPlugins" directory within the same directory that contains your :code:`plugin-GUI` repository: Then, using the command line or the `GitHub Desktop <https://github.com/apps/desktop/>`__ app, clone your the plugin repository into this new folder. Your directory structure should look something like this:

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

Inside the "Source" folder of the plugin repository you just cloned, you'll find the :code:`OpenEphysLib.cpp` file that contains critical information about your plugin. Open it in your preferred text editor and make the following changes, depending on the type of plugin you're creating.

**Processor** and **Visualizer** Plugins
-------------------------------------------

Most plugins will be "processors," meaning they implement the :code:`process()` method of `GenericProcessor.h <https://github.com/open-ephys/plugin-GUI/blob/main/Source/Processors/GenericProcessor/GenericProcessor.h>`__. This method is called repeatedly during the GUI's acquisition loop, so each plugin has a chance to respond to incoming data (or generate its own). If they include a canvas for displaying data, Processor plugins can also be Visualizer plugins.

Processor and Visualizer plugins should edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++

   // name of the library (since one OpenEphysLib file can be used for multiple plugins)
   info->name = "ProcessorPluginLibrary";

   // update this whenever you make changes to your plugin
   info->libVersion = "0.1.0"; 

   // edit to change how your plugin's name is displayed in the GUI
   info->processor.name = "Plugin Name"; 

   // Select one of the following: SOURCE, FILTER, or SINK
   info->processor.type = Plugin::Processor::FILTER; 

   // Replace "ProcessorPlugin" with the class name of your plugin
   info->processor.creator = &(Plugin::createProcessor<ProcessorPlugin>);

**Data Thread** Plugins
------------------------

Plugins that acquire data in a separate thread (e.g., because the acquisition hardware is not sychronized with the computer's clock) should be implemented as a Data Thread plugin. Instead of calling the :code:`process()` method directly, DataThread plugins add data to a buffer as it becomes available, which will be automatically copied into the GUI's signal chain. 

Data Thread plugins should edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++

   // name of the library (since one OpenEphysLib file can be used for multiple plugins)
   info->name = "DataThreadLibrary";

   // update this whenever you make changes to your plugin
   info->libVersion = "0.1.0"; 
   
   // edit to change how your plugin's name is displayed in the GUI
   info->dataThread.name = "Plugin Name"; 

   // Replace "DataThreadClassName" with the class name of your plugin
   info->dataThread.creator = &createDataThread<DataThreadClassName>;

**Record Engine** Plugins
--------------------------------------------

Record Engines define new output formats for the GUI. By default, the GUI ships with one Record Engine, the :ref:`binaryformat`. Two additional Record Engines are available via the Plugin Installer (:ref:`openephysformat` and :ref:`nwbdataformat`).

If you're creating a new Record Engine, you'll need to modify the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++

   // name of the library (since one OpenEphysLib file can be used for multiple plugins)
   info->name = "RecordEngineLibrary";

   // update this whenever you make changes to your plugin
   info->libVersion = "0.1.0"; 
   
   // edit to change how your plugin's name is displayed in the GUI
   info->recordEngine.name = "Plugin Name"; 

   // Replace "DataThreadClassName" with the class name of your plugin
   info->dataThread.creator = &(Plugin::createRecordEngine<RecordEnginePlugin>);

**File Source** Plugins
--------------------------------------------

File Sources allow the :ref:`filereader` to stream data from various file formats. By default, the File Reader can load data saved in the :ref:`binaryformat`. Two additional File Sources are available via the Plugin Installer (:ref:`openephysformat` and :ref:`nwbdataformat`).

.. code-block:: c++

   // name of the library (since one OpenEphysLib file can be used for multiple plugins)
   info->name = "FileSourceLibrary";

   // update this whenever you make changes to your plugin
   info->libVersion = "0.1.0"; 
   
   // edit to change how your plugin's name is displayed in the GUI
   info->fileSource.name = "Plugin Name"; 

    // Semicolon-separated list of supported file extensions (don't include a "." for these)
   info->fileSource.extensions = "csv;json"; 

   // Replace "FileSourcePlugin" with the class name of your plugin
   info->dataThread.creator = &(Plugin::createFileSource<FileSourcePlugin>);

Adding source code
#################################

Add the plugin's source files to the "Source" directory, or use the included files as a starting point.

By default, CMake will add any files with **.h** or **.cpp** extensions that live in the "Source" directory. If you have files with alternate extensions, you'll have to edit the following line of **CMakeLists.txt**:

.. code-block::

   file(GLOB_RECURSE SRC_FILES LIST_DIRECTORIES false "${SOURCE_PATH}/*.cpp" "${SOURCE_PATH}/*.h")


Including external libraries
################################

If your plugin depends on external libraries, it is necessary to manually edit the **CMakeLists.txt** file. The relevant lines are commented out at the end of this file.


Compiling your plugin
#########################

Follow the instructions on :ref:`compilingplugins` to compile your new plugin and test it out inside the GUI.

|
