.. _compilingplugins:
.. role:: raw-html-m2r(raw)
   :format: html

Compiling plugins
#################################

Whether you're creating a new plugin or you cloned an existing plugin repository, the following steps will allow you to compile your plugin across all three platforms. Note that `CMake <https://cmake.org/>`__ is required in all cases. See the :ref:`compiling the GUI <compilingthegui>` page for recommended instructions for installing CMake if you don't have it already.

Most Open Ephys GUI plugins work equally well on Windows, Linux, and Mac. However, there are some plugins that rely on platform-specific libraries, and can only be compiled for certain platforms. Be sure to check the relevant :ref:`plugin documentation<plugins>` page for platform compatibility.

.. important:: These instructions assume you have already compiled the main application from source. If not, you should start by following the instructions on :ref:`this page <compilingthegui>`.


Obtaining the source code
--------------------------

The first step for compiling a pre-existing plugin is downloading the source code. If you're planning to make your own changes to the plugin, we recommend first forking the plugin's GitHub repository to your own account, then cloning the fork via the `command line <https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository#cloning-a-repository-using-the-command-linee>`__ or the `GitHub Desktop app <https://desktop.github.com/>`__. If you're not planning to make any changes to the plugin, you can clone the original repository or download the code as a **.zip** file.

All of the officially supported plugins assume that the plugin source code is contained within a separate directory *at the same level* as that of the host application (the :code:`plugin-GUI` GitHub repository). Before attempting to compile your plugin, make sure your directory structure looks something like this:

.. code-block::

   Code
   ├── plugin-GUI
   │   ├── Build
   │   ├── Source
   │   └── ...
   │
   ├── OEPlugins
   │   └── plugin-1
   │   │   ├── Build
   │   │   ├── Source
   │   │   └── ...
   │   │
   │   └── plugin-2
   │       ├── Build
   │       ├── Source
   │       └── ...


Windows
--------

Generate the Visual Studio 2022 project files by typing the following from the command prompt inside the plugin's top-level directory:

.. code-block:: bash

   > cd Build
   > cmake -G "Visual Studio 17 2022" -A x64 ..

.. note:: For earlier version of Visual Studio, substitute the last command with: |br| :code:`cmake -G "Visual Studio 16 2019" -A x64 ..` |br| :code:`cmake -G "Visual Studio 15 2017 Win64" ..` |br| :code:`cmake -G "Visual Studio 14 2015 Win64" ..` |br| or |br| :code:`cmake -G "Visual Studio 12 2013 Win64" ..`

Next, launch Visual Studio and open the :code:`OE_PLUGIN_<plugin-name>.sln` file that was just created in the "Build" folder. Select the appropriate configuration (Debug/Release) and either build the solution or build the :code:`ALL_BUILD` project. That will run the build process on all projects except :code:`INSTALL`, thus building the plugin.

Selecting the INSTALL project and manually building it will trigger the install procedure, copying the plugin and any required files, if any, to the GUI's appropriate directories.

macOS
--------

To create the Xcode project files for the plugin, type the following commands from the plugin's top-level directory:

.. code-block:: bash

   $ cd Build
   $ cmake -G "Xcode" ..

Note that the final two periods are critical for getting this command to work.

Next, launch Xcode and open the :code:`<plugin-name>.xcodeproj` file that now lives in the "Build" directory.

Building the :code:`ALL_BUILD` scheme will build the plugin, while selecting :code:`INSTALL` will install the plugin in the appropriate location within the :code:`plugin-GUI` Build directory.

.. important:: If you're building the plugin on a Mac with Apple Silicon, you'll need to make sure the :code:`ALL_BUILD` profile is set use "Rosetta". You will likely need to first set the build target to "Any Mac," and then select the "My Mac (Rosetta)" option that appears. It is possible to build a version of the GUI that runs natively on Apple Silicon, but there are a few extra steps involved, and it won't work with plugins downloaded via the Plugin Installer. If you're interested in this, please reach out to support@open-ephys.org for more info.

The default Xcode build configuration is "Debug." To build the plugin in "Release" mode either modify the scheme settings or, instead of clicking Project/Build to build and install the plugin select **Project > Build for > Profiling**.

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

Next, running:

.. code-block:: bash

   $ make install

will copy the plugin and any additional required files to the appropriate location within the host application.


.. |fork icon| image:: ../_static/images/developerguide/fork.svg
   :height: 15

.. |br| raw:: html

  <br/>
