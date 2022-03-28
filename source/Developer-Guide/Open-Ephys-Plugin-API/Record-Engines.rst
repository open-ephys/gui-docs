.. _recordengines:

.. default-domain:: cpp

Record Engines
=====================

The Open Ephys Plugin API consists of all of the classes and methods that can be used by plugins. The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated by the :code:`PLUGIN_API` macro. 

.. note:: This documentation is focused on "Processor" plugins, which inherit from the :code:`GenericProcessor` class. Not all of the methods described below are available to other types of plugins, such as DataThreads, RecordEngines, and FileSources.

Adding a new data format
========================

In the Open Ephys GUI, data formats are defined within plugins, making it straightforward to add compatibility with alternate formats.

The GUI currently supports four data formats:

* :ref:`binaryformat` - the default format, based on widely used standards such as NumPy and JSON files.

* :ref:`openephysformat` - the GUI's original data format, designed to be highly fault tolerant

* :ref:`nwbdataformat` - the 1.0 specification of the Neurodata Without Borders format (soon to be replaced by the 2.0 spec)

* :ref:`kwikdataformat` - the now-deprecated HDF5-based format. This will soon be removed from the GUI, so it's not recommended for use.

In order to add a new format for writing by the Record Node, you must define a **RecordEngine** plugin. To add a new format for reading by the File Reader, you must define a **FileSource** plugin. 

This page will outline the basic steps for creating a new data format plugin. These instructions assume you have already compiled the main application. If not, follow the instructions on :ref:`this page <compilingthegui>`.

Creating a new data format repository
########################################

The first step in creating a new data format is to create a repository from the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template repository.

3. Click the green "Use this template" button.

4. Choose a name for your plugin. It should succinctly capture the plugin's functionality (and doesn't need to include the word "plugin").

5. Click the green "Create repository from template" button.

On your local machine, create an "OEPlugins" directory within the same directory that contains your :code:`plugin-GUI` repository: Then, using the command line or the `GitHub Desktop <https://desktop.github.com/>`__ app, clone your the plugin repository into this new folder. Your directory structure should look something like this:

.. code-block:: 

   code_directory/
      plugin-GUI/
      OEPlugins/
         NewDataFormat/
            Source/
            Build/
            CMakeLists.txt
            CMAKE_README.txt
            README.md


Editing :code:`OpenEphysLib.cpp`
#####################################

Inside the "Source" folder of the plugin repository you just cloned, you'll find the :code:`OpenEphysLib.cpp` file that contains critical information about your data format. Open it in your preferred text editor and make the following changes, depending on the type of data format you're creating:

**RecordEngines** Plugins
---------------------------

Plugins that write data are **RecordEngines**, meaning they must implement the :code:`writeData()`, :code:`writeEvent()`, and :code:`writeSpike()` methods of `RecordEngine.h <https://github.com/open-ephys/plugin-GUI/blob/master/Source/Processors/RecordNode/RecordEngine.h>`__. These methods are called whenever there is new data to be written to disk. RecordEngine plugins should uncomment and edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++
   
   // specifies that we are creating a RecordEngine plugin
   info->type = Plugin::PLUGIN_TYPE_RECORD_ENGINE;

   // edit to change how your plugin's name is displayed in the GUI
   info->recordEngine.name = "Record Engine Name";

   // Replace "RecordEngineClassName" with the namespace and class name of your plugin
   info->recordEngine.creator = &(Plugin::createRecordEngine<RecordEngineClassName>);;

**FileSource** Plugins
------------------------

**FileSource** plugins define how data should be imported by the File Reader. These plugins must implement the :code:`readData()` method of `FileSource.h <https://github.com/open-ephys/plugin-GUI/blob/master/Source/Processors/FileReader/FileSource.h>` (among others). FileSource plugins add data to a buffer as it becomes available, which will be automatically copied into the GUI's signal chain. FileSource plugins should uncomment and edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++
   
   // specifies that we are creating a FileSource plugin
   info->type = Plugin::PLUGIN_TYPE_FILE_SOURCE;;

   // edit to change how your plugin's name is displayed in the GUI
   info->fileSource.name = "File Source Name";

   //Semicolon separated list of supported extensions. e.g.: "txt;dat;info;kwd"
   info->fileSource.extensions = "xxx;xxx;xxx"; 

   // Replace "FileSourceClassName" with the class name of your plugin
   info->fileSource.creator = &(Plugin::createFileSource<FileSourceClassName>);

.. note:: It's possible for a single repository to contain multiple plugins (e.g., both a RecordEngine and a FileSource for writing/reading a particular format). In this case, make sure you set the value of :code:`NUM_PLUGINS` to 2, and implement a separate :code:`case` statement for each plugin.


Adding source code
#################################

Add the data format's source files to the "Source" directory, or use the included files as a starting point.

If you're using the template files, choose the header and cpp files corresponding to the type of plugin you're creating (RecordEngine or FileSource, or both), and delete the rest. You'll have to find and replace the default class name with the name of your plugin's class.

By default, CMake will add any files with **.h** or **.cpp** extensions that live in the "Source" directory. If you have files with alternate extensions, you'll have to edit the following line of **CMakeLists.txt**:

.. code-block::

   file(GLOB_RECURSE SRC_FILES LIST_DIRECTORIES false "${SOURCE_PATH}/*.cpp" "${SOURCE_PATH}/*.h")


Including external libraries
################################

If your plugin depends on external libraries, it is necessary to manually edit the **CMakeLists.txt** file. The relevant lines are commented out at the end of this file.


Compiling your data format
############################

Follow the instructions on :ref:`compilingplugins` to build your new data format.
