.. _openephyspluginAPI:

.. default-domain:: cpp

Open Ephys Plugin API
=====================

The Open Ephys Plugin API consists of all of the classes and methods that can be used by plugins. The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated by the :code:`PLUGIN_API` macro.

There are five different types of plugins that can be created:

.. toctree::
    :maxdepth: 5

    Processor-Plugins
    Visualizer-Plugins
    Data-Threads
    Record-Engines 
    File-Sources

The unique API commands available for each type of plugin is detailed on their respective sub-pages.

