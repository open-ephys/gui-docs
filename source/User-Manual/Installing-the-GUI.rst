.. _installingthegui:
.. role:: raw-html-m2r(raw)
   :format: html

Installing the GUI
=====================

The Open Ephys GUI works equally well on Windows, Linux, and macOS, so which platform you choose is mostly up to you. However, there are some plugins (such as Neuropixels) that are currently only available for Windows. We recommend reading through the documentation of any :ref:`plugins` you plan to use to find out about potential limitations.

Windows
########

Via installer (recommended)
----------------------------

.. warning:: If you have previously installed version 1.x of the GUI and are an Open Ephys Acquisition Board user, you need to copy and paste the following into the File Explorer: :code:`%localappdata%\\Open Ephys\\shared-api10`, press enter to navigate to the folder, and delete the file :code:`libonidriver_ft600.dll`. Then, follow the instructions below to install the latest version of the GUI.

1. Click the "Windows" link on `this page`_ to download the installer for the latest version of the GUI.

2. Double-click the downloaded executable to launch the installer.

3. Follow the instructions within the installer and any additional windows related to driver installation.

4. The GUI will be installed in :code:`C:\\Program Files\\Open Ephys\\open-ephys.exe`.

.. _windows-via-zip:

Via .zip file
--------------

1. Download a **.zip** file for any previous release via this link: https://openephys.jfrog.io/ui/native/GUI-binaries/Release/windows/

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. If you're going to be using the GUI with an Open Ephys acquisition board, install the USB driver specific to your board by following the instructions `here <https://open-ephys.github.io/acq-board-docs/User-Manual/Driver-installation.html>`_.

4. Double-click the **open-ephys** executable to run the application.

.. note:: If this is your first time running the GUI on a particular machine, you will need to install the `Visual C++ v14 Redistributable package`_ before opening the application. This only applies to the .zip download, as this package will be added automatically when using the Windows installer.

Linux
######

Via installer (Ubuntu/Debian only)
-----------------------------------

1. Click the "Linux" link on `this page`_ to download the **.deb** file for the latest version of the GUI.

2. Double-click on the .deb file, and enter your password when prompted.

3. The GUI will be installed in :code:`/usr/local/bin/open-ephys-gui`.

4. To launch, click "Show Applications" and search for "Open Ephys GUI."

5. To uninstall, type :code:`sudo dpkg --remove open-ephys` in a terminal window.

.. _linux-via-zip:

Via .zip file (all distros)
----------------------------

1. Download a **.zip** file for any previous release via this link: https://openephys.jfrog.io/ui/native/GUI-binaries/Release/linux/

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys GUI"

3. If you're going to be using the GUI with an Open Ephys acquisition board, its permissions need to be configured as follows:

   a. Open a terminal and set your working directory to the folder you just renamed.

   b. Enter :code:`sudo cp 40-open-ephys.rules /etc/udev/rules.d` followed by your password.

   c. Enter :code:`sudo cp 51-ftd3xx.rules /etc/udev/rules.d` followed by your password.

   d. Enter :code:`service udev restart` on Ubuntu/Debian or :code:`sudo udevadm control --reload-rules` on Fedora/CentOS/Gentoo.

4. Double-click the **open-ephys** executable (or enter :code:`./open-ephys` in the terminal) to run the application.


macOS
######

Via DMG (recommended)
----------------------

1. Click the "macOS" link on `this page`_ to download the DMG file for the latest version of the GUI.

2. Double-click on the downloaded file to open it.

3. Drag **open-ephys** to the "Applications" folder.

4. Double-click on the **open-ephys** app bundle to launch the GUI.

.. _macos-via-zip:

Via .zip file
--------------

1. Download a **.zip** file for any previous release via this link: https://openephys.jfrog.io/ui/native/GUI-binaries/Release/mac/

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Double-click :code:`open-ephys` to run the application.


|
|

.. _this page: https://open-ephys.org/gui
.. _Visual C++ v14 Redistributable package: https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170

