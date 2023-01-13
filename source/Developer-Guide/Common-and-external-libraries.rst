.. _commonandexternallibraries:
.. role:: raw-html-m2r(raw)
   :format: html

Common and external libraries
================================

Plugins are the primary means of extending the GUI's functionality. While, some plugins don't need any external libraries to work, other plugins need specific libraries for it to work. The libraries or dependencies are essential as they add more functionality to plugins without having to write all the code behind it.

Some examples of plugins that use external libaries include:

* `Neuropixels PXI <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ (*Neuropixels API*)
* `ZMQ Interface <https://github.com/open-ephys-plugins/zmq-interface>`__ (*ZeroMQ*)

There are also some cases where you want to create your own library that can be used by multiple plugins or you want to wrap a library with your own custom changes to provide a standard interface for multiple plugins, then you can do so by creating a common library. A common library is a Open Ephys Plugin API specific library that can be used by any plugin built using the Plugin API. 

Some examples of common libraries and plugins that use them include:

* `OpenEphysHDF5Lib <https://github.com/open-ephys-plugins/OpenEphysHDF5Lib>`__ (*Common Library*) 
   * `NWB Format <https://github.com/open-ephys-plugins/nwb-format>`__
  

* `OpenEphysFFTW <https://github.com/open-ephys-plugins/OpenEphysFFTW>`__ (*Common Library*) 
   * `Phase Calculator <https://github.com/open-ephys-plugins/phase-calculator>`__ 
   * `Spectrum Viewer <https://github.com/open-ephys-plugins/spectrum-viewer>`__
  


This page will show you how to build common libraries and use external libraries. These instructions assume you have already compiled the main application. If not, follow the instructions on :ref:`this page <compilingthegui>`.


Creating a Common Library 
##########################

The first step in creating a new common library is to create a repository from the :code:`OECommonLib` template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to the `Common Library <https://github.com/open-ephys-plugins/OECommonLib>`__ template repository.

3. Click the green "Use this template" button.

4. Choose a name for your common library. It should succinctly capture the plugin's functionality.

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

The template repository already comes with the boilerplate code for the common library. You simply start writing your code after change all the class and file names to match your common library's name. 

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
   

.. note:: For every class you want to export to use in the plugins, you need to add the `COMMON_LIB` macro to the class declaration as done above. 

Setup plugins using common libraries
-------------------------------------

For all the plugins that are going to use you common library, we first need to copy the :code:`link_open_ephys_lib.cmake` script from the :code:`OECommonLib` template repo to the plugin's base directory. This script will find and link the common library that is specified when calling it. Then modify the plugin's :code:`CMakeLists.txt` file to include and run this script as follows:

.. code-block:: cmake

   include(link_open_ephys_lib.cmake)
   link_open_ephys_lib(${PLUGIN_NAME} <common_lib_name>)

Now, when you build & install the common library and then build plugin, it will find the installed common library and link to it. After that, installing the plugin and loading it into the GUI should load the common library symbols that the plugin needs.


External libraries
###################

External libraries can etiher be used by the plugin directlty, of one can create a common library that wraps such a library and can be used by multiple plugins. There are two types of external libraries supported by the Open Ephys Plugin API. They are:

Header-only / Class Libraries
------------------------------

These types of libraries are either a single header file or a set of C++ files with classes and functions defined in them. They do not need to be separately compiled, packaged and installed in order to be used. All that is required is to point the compiler at the location of the headers, and then #include the header files into the application source. Another advantage is that the compiler's optimizer can do a much better job when all the library's source code is available. They also have some drawbacks, including longer compilation time, any changes to the library requires recompilation of all the plugins dependent on it.

Examples of such libraries that are used by some the GUI or it's plugins include:

* `cpp-httplib <https://github.com/yhirose/cpp-httplib>`__ (header-only)
* `oscpack <https://code.google.com/archive/p/oscpack/>`__ (class-based)
  

Shared / dynamic Libraries
---------------------------

Shared or dynamic libraries are files linked against at compile time by the plugin or common library to load all the references included via the library's header and then distributed with the end-user application so that the application can load the library code at run time. This means, the plugin / common library need the external library files to build, as well as during runtime to load the library symbols. 

These library files are platform-specific. Windows requires a :code:`.lib` file during compile-time and :code:`.dll` file run-time, while Linux needs a "shared object" or a :code:`.so` file  and MacOS needs a :code:`.dylib` file for both compile and run time. Since Windows does not have standardized paths for libraries, as Linux and MacOS do, it is necessary to pack the appropriate Windows version of the required libraries alongside the source code files. For Linux and MacOS, you can either install these dependencies to the standardized paths using the OS's package manager or manually, or you can provide the library files alongside the source code files just like Windows. To allow the plugin / common libray to find and load these library files during compile-time and run-time, you also need to modify their :code:`CMakeLists.txt` file.

The steps for modifying the :code:`CMakeLists.txt` as well as providing and installing the libraries are as follows:

1. Grab the library files for each platform the plugin / common library supports and place them alongside the source code as follows:

.. code-block:: 

   OEPlugins
   └─ Plugin/Common_Lib
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



4. Then need to make sure that the plugin / common library is able to find the library files during compile time. This can be done in two different ways dependeing on the type of library. For most commonly used libraries, the :code:`find_package` option is recommended. An example would be

.. code-block:: cmake

   find_package(ZLIB)
   target_link_libraries(${COMMONLIB_NAME} ${ZLIB_LIBRARIES})
   target_include_directories(${COMMONLIB_NAME} PRIVATE ${ZLIB_INCLUDE_DIRS})

If there is no standard package finder for cmake, find_libraryand find_path can be used to find the library and include files respectively. The commands will search in a variety of standard locations, for example

.. code-block:: cmake

   #the different names after names are not a list of libraries to include, but a list of possible names the library might have, useful for multiple architectures. find_library will return the first library found that matches any of the names
   find_library(ZMQ_LIBRARIES NAMES libzmq-v120-mt-4_0_4 zmq zmq-v120-mt-4_0_4) 
   find_path(ZMQ_INCLUDE_DIRS zmq.h)

   target_link_libraries(${COMMONLIB_NAME} ${ZMQ_LIBRARIES})
   target_include_directories(${COMMONLIB_NAME} PRIVATE ${ZMQ_INCLUDE_DIRS})


5. Lastly, we need to make sure the plugin / common library is able to find the run-time at the location it expects i.e. GUI's :code:`shared` directory. To do that, we need to tell CMake to install the library's run-time files to the code:`shared` directory. This can be done by adding the following lines at the end of the :code:`CMakeLists.txt` file

.. code-block:: cmake

   if (MSVC)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/windows/bin/${CMAKE_LIBRARY_ARCHITECTURE}/ DESTINATION ${GUI_BIN_DIR}/shared)
   elseif(LINUX)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/linux/bin/ DESTINATION ${GUI_BIN_DIR}/shared)
   elseif(APPLE)
      install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/libs/macos/bin/ DESTINATION $ENV{HOME}/Library/Application\ Support/open-ephys/shared-api8)
   endif()

6. Now, when you run CMake and build the plugin / common library, it should be able to find and load the library during compile-time and while installing the plugin, it will automatically install the run-time library at the correct location.


