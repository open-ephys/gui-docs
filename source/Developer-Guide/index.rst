.. _developerguide:

Developer Guide
===============

This section is meant to help anyone interested in modifying the source code for the Open Ephys GUI, or extending its functionality via plugins. It can also be helpful to read if you simply want to gain a better understanding of the inner workings of the software. 

No matter your level of experience, there are a number of ways that you can help improve the GUI:

Reporting bugs
---------------

If you observe any unexpected behavior with the GUI, please `submit an issue on GitHub <https://github.com/open-ephys/plugin-GUI/issues>`__. This is the easiest way for us to keep track of things that need to be fixed. 

When you submit an issue, make sure to include what version of the GUI you're using (visible in the lower right of the "Graph" tab), and what operating system you're running it on.

Building a new plugin
----------------------

The recommended way to add features to the GUI is by building a new plugin. Plugins allow the GUI to interface with new data sources, trigger events on specific neural states, display data within rich visualizations, or write data to a new format, among many other possibilities. Officially supported plugins are listed in the User Manual :ref:`User Manual<plugins>`, and a list of additional community-generated plugins is maintained `her <https://open-ephys.atlassian.net/wiki/display/OEW/Third-party+plugin+repositories>`__.

Before you :ref:`create a new plugin<creatinganewplugin>`, you'll need to have some familiarity with C++, as well as makefiles (Linux), Xcode (macOS), or Visual Studio (Windows).

Updating the host application
------------------------------

If you'd like to make changes to the Open Ephys GUI main repository, first read through :ref:`this section<modifyingthehostapplication>` of the developer documentation. Then, take a look at our list of `active projects <https://github.com/open-ephys/plugin-GUI/projects>`__ to find out about the features we're planning to update in the near future. If you're interested in tackling one of these projects, or have other ideas for useful additions to the main application, don't hesistate to `open an issue on GitHub <https://github.com/open-ephys/plugin-GUI/issues>`__.

Other projects
---------------

If you are interested in helping out with longer-term projects, or if you're able to donate developer time, please get in touch via `info@open-ephys.org <mailto:info@open-ephys.org>`__.


.. toctree::
    :hidden:
    :maxdepth: 5

    Compiling-the-GUI
    Compiling-plugins
    Creating-a-new-plugin
    Adding-a-new-data-format
    Open-Ephys-Plugin-API
    Distributing-plugins
    Modifying-the-host-application


.. role:: raw-html-m2r(raw)
   :format: html


