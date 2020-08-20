.. _installingthegui:
.. role:: raw-html-m2r(raw)
   :format: html

Installing the GUI
=====================

The Open Ephys GUI works equally well on Windows, Linux, and macOS, so which platform you choose is mostly up to you. However, there are some plugins (such as Neuropixels) that are currently only available for Windows. We recommend reading through the documentation of any :ref:`plugins` you plan to use to find out about potential limitations.

Windows
########
1. Click the "Windows" link on `this page`_ to download the ZIP file.

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Run :code:`FrontPanelUSB-DriverOnly-4.4.0.exe` to install the Open Ephys acquisition board driver. 

4. Double-click the :code:`open-ephys` executable to run the application.

.. note:: If the GUI won't open due to a missing DLL, you'll need to install the `Visual Studio 2015 2017 and 2019 redistributable package`_.

Linux
######

1. Click the "Linux" link on `this page`_ to download the ZIP file.

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Open a terminal and set your working directory to the folder you just renamed.

4. Enter :code:`sudo cp 40-open-ephys.rules /etc/udev/rules.d` followed by your password.

5. Enter :code:`service udev restart` on Ubuntu/Debian or :code:`sudo udevadm control --reload-rules` on Fedora/CentOS to allow the GUI to communicate with the Open Ephys acquisition board.

6. Double-click the :code:`open-ephys` executable (or enter :code:`./open-ephys` in the terminal) to run the application.


macOS
######

1. Click the "macOS" link on `this page`_ to download the ZIP file.

2. Drag and drop the downloaded and unzipped folder to the location of your choice, and rename it "Open Ephys"

3. Double-click :code:`open-ephys` to run the application.

.. note:: Depending on your security settings, you may see a message announcing that "'open-ephys' cannot be opened because the developer cannot be verified." If so, go to System Prefererences > Security and Privacy > General and click "Open Anyway" to allow the GUI to launch.

|
|

.. _this page: https://open-ephys.org/gui
.. _Visual Studio 2015 2017 and 2019 redistributable package: https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads

