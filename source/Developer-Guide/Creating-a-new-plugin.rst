.. _creatinganewplugin:
.. role:: raw-html-m2r(raw)
   :format: html

Creating and compiling plugins
================================

Plugins are the primary means of extending the GUI's functionality. You can create "source" plugins that interface with new types of data acquisition hardware, "filter" plugins that modify incoming data, or "sink" plugins that send data to other applications or output devices. "Record engines" are also plugins that define new formats for reading an writing data.

Plugins can either live inside the main `plugin-GUI <https://github.com/open-ephys/plugin-GUI>`__, or in their own repository. Hosting plugins in separate repositories is recommended, so they do not introduce additional dependencies and can be updated separately from the main application.

Some examples of externally hosted plugins include:

* `Neuropixels PXI <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ (Source)
* `Stream Muxer <https://github.com/open-ephys-plugins/StreamMuxer>`__ (Filter)
* `Probe Viewer <https://github.com/open-ephys-plugins/probe-viewer>`__ (Sink)
* `NWB Format <https://github.com/open-ephys-plugins/NWBFormat>`__ (Record Engine)

This page will show you how to create new plugins and compile them for Linux, Mac, and Windows. These instructions assume you have already compiled the main application. If not, follow the instructions on :ref:`this page <compilingthegui>`.

Creating a new plugin repository
#################################

The first step in creating a new plugin is to create a repository from the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template.

1. Log in to your `GitHub <https://github.com/>` account.

2. Browse to https://github.com/open-ephys-plugins/OEPlugin 

3. Click the green "Use this template" button.

4. Choose a name for your plugin. It should succinctly capture the plugin's functionality (and doesn't need to include the word "plugin").

5. Click the green "Create repository from template" button.

Next, using the command line or the `GitHub Desktop <https://desktop.github.com/`__ app, clone your new repository into the same top-level directory that contains your :code:`plugin-GUI` repository:

.. code-block:: 

   /code_directory
      /plugin-GUI
      /NewPlugin

Adding plugin source code
#################################

Inside the directory that was just created, change the name of the "Plugin" directory to the name of your plugin, e.g. "NewPlugin."

Add the source files (or use the included files as a starting point).

Edit the OpenEphysLib.cpp file.

Compiling plugins
#################################

Whether you're creating a new plugin, or you cloned an existing plugin repository, the following steps will allow you to compile your plugin across all three platforms. Note that `CMake <https://cmake.org/>`__ is required in all cases. See the :ref:`compiling the GUI <compilingthegui>` page for recommended instructions for installing CMake if you don't have it already.


Windows
--------

Generate the Visual Studio project files by typing the following from the command prompt inside the :code:`plugin-GUI` top-level directory:

.. code-block:: bash

   > cd Build
   > cmake -G "Visual Studio 16 2019" -A x64 ..

.. note:: For earlier versions of Visual Studio, substitute the last command with: |br| :code:`cmake -G "Visual Studio 12 2013 Win64" ..` |br| :code:`cmake -G "Visual Studio 14 2015 Win64" ..` |br| or |br| :code:`cmake -G "Visual Studio 15 2017 Win64" ..`

Next, launch Visual Studio and open the :code:`newplugin.sln` file that was just created in the "Build" folder. Select the appropriate configuration (Debug/Release) and either build the solution or build the :code:`ALL_BUILD` project. That will run the build process on all projects except :code:`INSTALL`, thus building the plugin.

Selecting the INSTALL project and manually building it will trigger the install procedure, copying the plugin and any required files, if any, to the GUI's appropriate directories.

macOS
--------

To create the Xcode project files for the plugin, type the following commands from the plugin's top-level directory:

.. code-block:: bash

   $ cd Build
   $ cmake -G "Xcode" ..

Note that the final two periods are critical for getting this command to work.

Next, launch Xcode and open the :code:`newplugin.xcodeproj` file that now lives in the "Build" directory.

Building the ALL_BUILD scheme will build the plugin while selecting the INSTALL in the scheme selector and Building it will install the plugin.

The default build configuration is Debug. To build the plugin in Release mode either modify the scheme settings or, instead of clicking Project/Build to build and install the plugin select Project/Build for/Profiling

Linux
--------

Generate the Linux makefiles by entering the following from the plugin's top-level directory:

.. code-block:: bash

   $ cd Build
   $ cmake -G "Unix Makefiles" ..

.. note:: To specify "Debug" or "Release" mode, add :code:`-DCMAKE_BUILD_TYPE=Release` or :code:`-DCMAKE_BUILD_TYPE=Debug` to the last command, just before the two periods. Setting a variable using a :code:`-D` argument will be permanent, with following calls to :code:`cmake` in the same folder using its set value even if the argument is not used in them. Variables can be either set to a different value by calling cmake with a different :code:`-D` option (thereby overwriting the existing value) or unset by calling :code:`cmake -UVARIABLE`.

Once the makefile generation step is finished, enter the following line from the "Build" directory:

.. code-block:: bash

   $ make

This will build the plugin.

Now, running:

.. code-block:: bash

   $ make install

will copy the plugin and any additional required files to the appropriate location within the host application.

Now, you can 

Debugging plugins
#################################

How can we debug plugins in external repositories?


.. |br| raw:: html



