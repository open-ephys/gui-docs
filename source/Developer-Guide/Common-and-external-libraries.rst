.. _commonandexternallibraries:
.. role:: raw-html-m2r(raw)
   :format: html

Common and external libraries
====================================

Not only do plugins make it easier to share new features with other users, they can also take advantage of a wide array of C++ libraries without adding new dependencies to the host application. While there are many plugins that only rely on the Open Ephys Plugin API, JUCE, and the C++ standard library, in some cases it is essential to call functions from more specialized libraries. External libraries can add powerful features with minimal extra code.

Some examples of plugins that use external libaries include:

* `Neuropixels PXI <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ (*Neuropixels API*)
* `ZMQ Interface <https://github.com/open-ephys-plugins/zmq-interface>`__ (*ZeroMQ*)

It's also possible to create a custom library that can be used by multiple plugins, or to wrap an existing library in a way that makes it easier to use with the Open Ephys GUI. These "Common Libraries" are Open Ephys-specific libraries that can be used by any plugin built using the Plugin API.

Some examples of common libraries and plugins that use them include:

* `OpenEphysHDF5Lib <https://github.com/open-ephys-plugins/OpenEphysHDF5Lib>`__ (*Common Library*) 
   * `NWB Format <https://github.com/open-ephys-plugins/nwb-format>`__
  

* `OpenEphysFFTW <https://github.com/open-ephys-plugins/OpenEphysFFTW>`__ (*Common Library*) 
   * `Phase Calculator <https://github.com/open-ephys-plugins/phase-calculator>`__ 
   * `Spectrum Viewer <https://github.com/open-ephys-plugins/spectrum-viewer>`__
  

This page will demonstrate how to use common libraries and external libraries in your plugins. These instructions assume you have already compiled the Open Ephys host application. If you haven't done that yet, follow the instructions on :ref:`this page <compilingthegui>`.


Common libraries
##########################

The first step in creating a new common library is to create a repository from the :code:`OECommonLib` template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to the `Common Library <https://github.com/open-ephys-plugins/OECommonLib>`__ template repository.

3. Click the green "Use this template" button and choose the "Create a new repository" option.

4. Choose a name for your common library. It should succinctly capture the library's functionality.

5. Click the green "Create repository from template" button.

On your local machine, create an "OEPlugins" directory within the same directory that contains your :code:`plugin-GUI` repository: Then, using the command line or the `GitHub Desktop <https://desktop.github.com/>`__ app, clone your the common library repository into this new folder. Your directory structure should look something like this:

.. code-block:: 

   OEPlugins
   └─ CommonLib
      ├─ Build/
      ├─ Source/
      ├─ CMakeLists.txt
      ├─ CMAKE_README.txt
      ├─ link_open_ephys_lib.cmake
      └─ README.md


Modifying the source code
--------------------------

The template repository already comes with the boilerplate code for the common library. You can start writing your code after changing all the class and file names to match your common library's name. 

.. code-block:: c++
   :caption: CommonLib.h

   #ifndef COMMONLIB_H_INCLUDED
   #define COMMONLIB_H_INCLUDED

   #include <CommonLibHeader.h>

   namespace CommonLibrary
   {
      class COMMON_LIB LibraryClass
      {
         public: 
            LibraryClass();
            ~LibraryClass();
      };
   }

   #endif


.. code-block:: c++
   :caption: CommonLib.cpp

   #include "CommonLib.h"

   using namespace CommonLibrary;

   LibraryClass::LibraryClass()
   {
   }

   LibraryClass::~LibraryClass()
   {
   }
   

.. note:: For every class you want to export for use by plugins, you need to add the `COMMON_LIB` macro to the class declaration as demonstrated above. 

Using the common library in a plugin
-------------------------------------

For all the plugins that are going to use you common library, we first need to copy the :code:`link_open_ephys_lib.cmake` script from the :code:`OECommonLib` template repo to the plugin's base directory. This script will find and link the common library that is specified when calling it. Then modify the plugin's :code:`CMakeLists.txt` file to include and run this script as follows:

.. code-block:: cmake

   include(link_open_ephys_lib.cmake)
   link_open_ephys_lib(${PLUGIN_NAME} <common_lib_name>)

Now, when you build & install the common library and then build plugin, it will find the installed common library and link to it. After that, installing the plugin and loading it into the GUI should load the common library symbols that the plugin needs.


External libraries
###################

Header-only / Class Libraries
------------------------------

These types of libraries are either a single header file or a set of C++ files with classes and functions defined in them. They do not need to be separately compiled, packaged, and installed in order to be used. All that is required is to copy the library files into your plugin's source code directory (usually in a sub-folder), point the compiler to the location of the headers, and then :code:`#include` the header files in any plugin source files that need to access the associated classes. Besides being easy to add to your plugin, header-only and class libraries make it possible for the compiler to optimize the code more effectively. They do have some drawbacks, however, including duplication of code, longer compilation time, and the fact that any changes to the library requires recompilation of all the source files that depend on it.

Examples of such libraries that are used by some the GUI or its plugins include:

* `cpp-httplib <https://github.com/yhirose/cpp-httplib>`__ (header-only)
* `oscpack <https://code.google.com/archive/p/oscpack/>`__ (class-based)
  

Shared / dynamic Libraries
---------------------------

Shared or dynamic libraries are pre-built libraries that are linked to the plugin or common library at compile time. The plugin / common library needs access to the library's header files, plus a library linker file that's only used for compilation. The plugin / common library must ship with a separate dynamic library, which is called by the plugin / common library at runtime.

These library files are platform-specific. Windows requires a :code:`.lib` file during compile-time and :code:`.dll` file at runtime. Linux needs a "shared object" or :code:`.so` file and macOS needs a :code:`.dylib` file for both compile and run time. Since Windows does not have standardized paths for libraries, as Linux and macOS do, it is necessary to pack the appropriate Windows version of the required libraries alongside the source code files. For Linux and macOS, you can either install these dependencies to the standardized paths manually or using a package manager, or you can provide the library files alongside the source code files just like Windows. To allow the plugin / common libray to find and load these library files during compile-time and runtime, you also need to modify their :code:`CMakeLists.txt` file.

The steps for modifying the :code:`CMakeLists.txt` as well as providing and installing the libraries are as follows:

1. Grab the library files for each platform the plugin / common library supports and place them alongside the source code as follows:

.. code-block:: 

   OEPlugins
   └─ Plugin_or_Common_Lib
      ├─ Build
      ├─ libs
      │   ├─ linux
      │   │   ├─ include           # library headers
      │   │   ├─ lib               # compile-time (.so) file(s)
      │   │   └─ bin               # runtime (.so.x) file(s)
      │   ├─ macos
      │   │   ├─ include           # library headers
      │   │   ├─ lib               # compile-time (.dylib) file(s)
      │   │   └─ bin               # runtime (.x.dylib) file(s)
      │   └─ windows
      │      ├─ include           # library headers
      │      ├─ lib               # compile-time (.lib) file(s)
      │      └─ bin               # runtime (.dll) file(s)
      └─ Source


2. Once you have you library files in place, open the plugin / common lib's  :code:`CMakeLists.txt` file

3. Now, we need to make sure that CMake is able to find the library files at the path above. To do so, we need to define the :code:`CMAKE_PREFIX_PATH` as follows

.. code-block:: cmake

   if(MSVC)
   
      set(CMAKE_PREFIX_PATH ${CMAKE_CURRENT_SOURCE_DIR}/libs/windows)

   elseif(LINUX)
      
      set(CMAKE_PREFIX_PATH ${CMAKE_CURRENT_SOURCE_DIR}/libs/linux)

   elseif(APPLE)
      
      set(CMAKE_PREFIX_PATH ${CMAKE_CURRENT_SOURCE_DIR}/libs/macos)

   endif()



4. We then need to make sure that the plugin / common library is able to find the library files during compile time. This can be done in two different ways dependeing on the type of library. For most commonly used libraries, the :code:`find_package` option is recommended. An example would be

.. code-block:: cmake

   find_package(ZLIB)
   target_link_libraries(${COMMONLIB_NAME} ${ZLIB_LIBRARIES})
   target_include_directories(${COMMONLIB_NAME} PRIVATE ${ZLIB_INCLUDE_DIRS})

If there is no standard package finder for CMake, :code:`find_library`` and :code:`find_path`` can be used to find the library and include files respectively. The commands will search in a variety of standard locations, for example:

.. code-block:: cmake

   #the different names after names are not a list of libraries to include, but a list of possible names the library might have, useful for multiple architectures. find_library will return the first library found that matches any of the names
   find_library(ZMQ_LIBRARIES NAMES libzmq-v120-mt-4_0_4 zmq zmq-v120-mt-4_0_4) 
   find_path(ZMQ_INCLUDE_DIRS zmq.h)

   target_link_libraries(${COMMONLIB_NAME} ${ZMQ_LIBRARIES})
   target_include_directories(${COMMONLIB_NAME} PRIVATE ${ZMQ_INCLUDE_DIRS})


5. Lastly, we need to make sure the plugin / common library is able to find the runtime library at the location it expects, i.e. the GUI's :code:`shared` directory. To do that, we need to tell CMake to install the library's runtime files to the :code:`shared` directory. This can be done by adding the following lines at the end of the :code:`CMakeLists.txt` file

.. code-block:: cmake

   if (MSVC)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/windows/bin/${CMAKE_LIBRARY_ARCHITECTURE}/ DESTINATION ${GUI_BIN_DIR}/shared)
   elseif(LINUX)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/linux/bin/ DESTINATION ${GUI_BIN_DIR}/shared)
   elseif(APPLE)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/macos/bin/ DESTINATION $ENV{HOME}/Library/Application\ Support/open-ephys/shared-api8)
   endif()

6. Now, when you run CMake and build the plugin / common library, it should be able to find and load the library during compile-time. When installing the plugin, it will automatically install the runtime library at the correct location.


