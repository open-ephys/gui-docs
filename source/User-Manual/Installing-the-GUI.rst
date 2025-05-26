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

1. Click the "Windows" link on `this page`_ to download the installer for the latest version of the GUI.

2. Double-click the downloaded executable to launch the installer.

3. Follow the instructions within the installer.

4. The GUI will be installed in :code:`C:\\Program Files\\Open Ephys\\open-ephys.exe`.

.. _windows-via-zip:

Via .zip file
--------------

1. Download a **.zip** file for any previous release via this link: https://openephys.jfrog.io/ui/native/GUI-binaries/Release/windows/

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Run :code:`FrontPanelUSB-DriverOnly-4.4.0.exe` to install the Open Ephys acquisition board driver. 

4. Double-click the **open-ephys** executable to run the application.

.. note:: If this is your first time running the GUI on a particular machine, you will need to install the `Visual Studio 2015, 2017, 2019, and 2022 redistributable package`_ before opening the application. This only applies to the .zip download, as this package will be added automatically when using the Windows installer.

Linux
######

Via installer (Ubuntu only)
-----------------------------

1. Click the "Linux" link on `this page`_ to download the **.deb** file for the latest version of the GUI.

2. Double-click on the .deb file, and enter your password when prompted.

3. The GUI will be installed in :code:`/usr/local/bin/open-ephys-gui`.

4. Type :code:`open-ephys` to launch the GUI.

5. To uninstall, type :code:`sudo dpkg --remove open-ephys` in a terminal window.

.. _linux-via-zip:

Via .zip file (all distros)
----------------------------

1. Download a **.zip** file for any previous release via this link: https://openephys.jfrog.io/ui/native/GUI-binaries/Release/linux/

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Open a terminal and set your working directory to the folder you just renamed.

4. Enter :code:`sudo cp 40-open-ephys.rules /etc/udev/rules.d` followed by your password.

5. Enter :code:`service udev restart` on Ubuntu/Debian or :code:`sudo udevadm control --reload-rules` on Fedora/CentOS/Gentoo to allow the GUI to communicate with the Open Ephys acquisition board.

6. Double-click the **open-ephys** executable (or enter :code:`./open-ephys` in the terminal) to run the application.


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
.. _Visual Studio 2015, 2017, 2019, and 2022 redistributable package: https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170

