.. _visualizerplugins:

.. default-domain:: cpp

Visualizer Plugins
=====================

The Open Ephys Plugin API consists of all of the classes and methods that can be used by plugins. The Plugin API defines how plugins can interact with the rest of the GUI, and also provides some convenient functions for building user interfaces and processing data. Classes and methods that are part of the Plugin API are indicated by the :code:`PLUGIN_API` macro. 

.. note:: This documentation is focused on "Processor" plugins, which inherit from the :code:`GenericProcessor` class. Not all of the methods described below are available to other types of plugins, such as DataThreads, RecordEngines, and FileSources.
