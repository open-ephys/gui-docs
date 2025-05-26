.. _compilingthegui:
.. role:: raw-html-m2r(raw)
   :format: html

Compiling the GUI
=====================

If you'd like to build the GUI from source, you'll have to download the code to your local machine. We recommend first forking the `GUI's GitHub repository <https://github.com/open-ephys/plugin-GUI>`__ to your own account, then cloning the fork via the `command line <https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository?tool=cli>`__ or the `GitHub Desktop app <https://github.com/apps/desktop/>`__. 


Windows
#######

Building the GUI on Windows requires Visual Studio (the free Community Edition is fine). We recommend using `Visual Studio 2022 Community <https://visualstudio.microsoft.com/downloads/>`__, but it will also work with VS2013, 2015, 2017, or 2019.

You'll also need to install `CMake <https://cmake.org/download/>`__ from the official download page. During installation, we recommend selecting the option  to *"Add CMake to the system PATH"* for either all users or current users, to allow easy use of the CMake command-line tools.

Once you have CMake installed, you can generate the Visual Studio project files by typing the following from the command prompt inside the :code:`plugin-GUI` top-level directory:

.. code-block:: bash

   > cd Build
   > cmake -G "Visual Studio 17 2022" -A x64 ..

.. note:: For earlier version of Visual Studio, substitute the last command with: |br| :code:`cmake -G "Visual Studio 16 2019" -A x64 ..` |br| :code:`cmake -G "Visual Studio 15 2017 Win64" ..` |br| :code:`cmake -G "Visual Studio 14 2015 Win64" ..` |br| or |br| :code:`cmake -G "Visual Studio 12 2013 Win64" ..`

Next, launch Visual Studio and open the :code:`open-ephys-GUI.sln` file that was just created in the "Build" folder.

To compile the GUI, select "Build > Build Solution" (shortcut key: F6), or press the "Debug" button to build and run. This will build the main application, the :code:`open-ephys.lib` file, and all of the plugins that are included in the main repository.

.. tip:: In the "Solution Explorer," right-click on **open-ephys** and select "Set as Startup Project" to automatically launch the GUI when you start the debugger. 


macOS
#####

Building the GUI on a Mac requires `Xcode <https://developer.apple.com/xcode/>`__. This can be installed via the App Store. You'll also need to install the Xcode command-line tools by entering the following line in a terminal window:

.. code-block:: bash

   $ command xcode-select --install

In order generate the build files for the GUI, you'll need to install `CMake <https://cmake.org/download/>`__ for Mac OS X 10.7 or later. Once that's installed, make the :code:`cmake` command-line tools accessible system-wide by entering the following line in a terminal window: 

.. code-block:: bash

   $ sudo "/Applications/CMake.app/Contents/bin/cmake-gui" --install

Now, you're ready to build the GUI! To create the Xcode project files, type the following commands from the :code:`plugin-GUI` top-level directory:

.. code-block:: bash

   $ cd Build
   $ cmake -G "Xcode" ..

Note that the final two periods are critical for getting this command to work.

Next, launch Xcode and open the :code:`open-ephys-GUI.xcodeproj` file that now lives in the "Build" directory.

Inside XCode, hit the "Run" button to build the GUI and plugins from source. To run the GUI from within XCode, select the "open-ephys" scheme from the drop-down menu next to the Run and Stop buttons, instead of "ALL_BUILD".

Linux
######

In order generate the build files for the GUI, you'll need to install `CMake <https://cmake.org/download/>`__ for Linux. On Ubuntu and other Debian-based distributions, you can do so with the following commands:

.. code-block:: bash

   $ sudo apt-get update
   $ sudo apt-get install cmake

Next, install the Linux dependencies by entering the following line from the :code:`plugin-GUI` top-level directory:

.. code-block:: bash

   $ sudo Resources/Scripts/install_linux_dependencies.sh

If you want the GUI to be able to communicate with the Open Ephys acquisition board, you'll also need to enter the following lines:

.. code-block:: bash
   
   $ sudo cp Resources/Scripts/40-open-ephys.rules /etc/udev/rules.d
   $ sudo service udev restart

.. note:: On Fedora/CentOS distros, substitute :code:`sudo udevadm control --reload-rules` for the last command

Now, generate the Linux makefiles by entering:

.. code-block:: bash

   $ cd Build
   $ cmake -G "Unix Makefiles" ..

.. note:: To specify "Debug" or "Release" mode, add :code:`-DCMAKE_BUILD_TYPE=Release` or :code:`-DCMAKE_BUILD_TYPE=Debug` to the last command, just before the two periods. Setting a variable using a :code:`-D` argument will be permanent, with following calls to :code:`cmake` in the same folder using its set value even if the argument is not used in them. Variables can be either set to a different value by calling cmake with a different :code:`-D` option (thereby overwriting the existing value) or unset by calling :code:`cmake -UVARIABLE`.

Once the makefile generation step is finished, enter the following line from the "Build" directory:

.. code-block:: bash

   $ make

This will build the main application as well as the plugins. If this step is successful, there will be a compiled binary at :code:`Build/Debug/open-ephys` or :code:`Build/Release/open-ephys`.

.. |fork icon| image:: ../_static/images/developerguide/fork.svg
   :height: 15

.. |br| raw:: html

  <br/>