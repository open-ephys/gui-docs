.. _howtomakeyourownplugin:
.. role:: raw-html-m2r(raw)
   :format: html

How To Make Your Own Plugin
============================

The Open Ephys GUI's plugin architecture allows it to be expanded with external modules (plugins) that can be developed independently of the main application. This is the primary way which we encourage users to add new functionality to the GUI.  

This tutorial will guide you through the steps of making a plugin from scratch by creating a "TTL Event Generator" plugin. :code:`TTL` events are one of three types of events that are supported by the GUI (along with :code:`TEXT` and :code:`BINARY` events). TTL events represent ON/OFF transitions that are traditionally associated with "Transistor-Transitor Logic" circuits. Within the GUI, TTL events are more general, and can also be generated in software. This plugin will add TTL events to a data stream, either at a specified frequency, or whenever a trigger button is pressed.

Along with explaining how to configure the plugin and set up the main :code:`process()` method, this tutorial will demonstrate how to create UI components for the plugin using the underlying `JUCE API <https://juce.com/>`__. These instructions assume you have already compiled the main application from source. If not, you should start by following the instructions on :ref:`this page <compilingthegui>`.

Creating a new plugin repository
#################################

The first step in creating a new plugin is to create a repository from the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template.

1. Log in to your `GitHub <https://github.com/>`__ account.

2. Browse to the `OEPlugin <https://github.com/open-ephys-plugins/OEPlugin>`__ template repository.

3. Click the green "Use this template" button.

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-01.png
  :alt: OEPlugin Template Repository

4. Since the plugin will generate TTL events, lets name the repository as "TTLEventGenerator"

5. Click the green "Create repository from template" button.

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-02.png
  :alt: Create TTLEventGenerator Repository

On your local machine, create an "OEPlugins" directory within the same directory that contains your :code:`plugin-GUI` repository: Then, using the command line or the `GitHub Desktop <https://desktop.github.com/>`__ app, clone the newly created plugin repository into this new folder. Your directory structure should look something like this:

.. code-block:: 

   code_directory/
      plugin-GUI/
      OEPlugins/
         TTLEventGenerator/
            Source/
            Build/
            CMakeLists.txt
            CMAKE_README.txt
            README.md


Removing unncessary files
#################################

Since :code:`TTL Events` is going to be a "Filter" processor, remove all the files from the :file:`Source` directory except :file:`OpenEphysLib.cpp`, :file:`ProcessorPlugin.cpp`, and :file:`ProcessorPlugin.h` files.

Editing :code:`OpenEphysLib.cpp` and other files
#################################################

Inside the "Source" directory, you'll find the :file:`OpenEphysLib.cpp` file that contains critical information about your plugin. Open it in your preferred text editor and make the following changes:

TTLEventGenerator plugin will be a "processor", meaning it implements the :code:`process()` method of the `GenericProcessor <https://github.com/open-ephys/plugin-GUI/blob/master/Source/Processors/GenericProcessor/GenericProcessor.h>`__ class. This method is called repeatedly during the GUI's acquisition loop, so each plugin has a chance to respond to incoming data (or, in this case, generate its own data). 

To specify that this is a processor plugin, uncomment and edit the following lines in :code:`OpenEphysLib.cpp`:

.. code-block:: c++
   
   // specifies that we are creating a Processor plugin
   info->type = PluginType::PLUGIN_TYPE_PROCESSOR;

   // edit to change how the plugin's name is displayed in the GUI
   info->processor.name = "TTL Events"; 

   // Select the type of processor
   info->processor.type = ProcessorType::FilterProcessor; 

   // Replace "ProcessorPluginSpace::ProcessorPlugin" with the class name of TTL Events plugin
   info->processor.creator = &(Plugin::createProcessor<TTLEventGenerator>);

|
| Then, rename the :code:`ProcessorPlugin.cpp` & :code:`ProcessorPlugin.h` files to :code:`TTLEventGenerator.cpp` and :code:`TTLEventGenerator.h`, and find and replace the **ProcessorPlugin** class name with **TTLEventGenerator** in the .cpp and .h files.

Compiling your plugin
########################

At this point, you should be able to compile your plugin and load it into the GUI.

TODO: Add compilation instructions
-----------------------------------

Implementing the :code:`process()` method
##########################################

Right now, our plugin doesn't have any effect within the signal chain. Data passed into the :code:`process()` method will not be altered in any way, nor will any events be added to the data stream.

Let's change that by inserting code to add a TTL ON and OFF events at an interval of 1 second. For now, we will hard-code the relevant parameters. In the subsequent steps, we will make it possible to change these parameters via UI elements in the plugin's editor.

Before we can add events during acquisition, we need to announce to downstream processors that this plugin is capable of generating its own events. In the plugin's header file, add the following function declarations, plus some class members:

.. code-block:: c++

   void createEventChannels() override;

   void startAcquisition() override;

   EventChannel* eventChannel; // pointer to our event channel

   int sampleRate; // holds the sample rate for incoming data

   int counter; // counts the total number of incoming samples
   bool state; // holds the channel state (on or off)

This will allow us to override the default implementation of the :code:`createEventChannels()` method, which is automatically called whenever a plugin needs to update its settings.

Next, in the .cpp file, add the implementation:

.. code-block:: c++

   void TTLEventGenerator::createEventChannels()
   {

      sampleRate = (int) getSampleRate(0);

      const DataChannel* inputChannel = getDataChannel(0);

      if (!inputChannel) // no input channels to this plugin
      {
            eventChannel = new EventChannel(EventChannel::TTL, // channel type
                                             8, // number of sub-channels
                                             1, // data packet size
                                             sampleRate, // sampleRate
                                             this) // source processor
      } else {
         eventChannel = new EventChannel(EventChannel::TTL, // channel type
                                             8, // number of sub-channels
                                             1, // data packet size
                                             inputChannel, // pointer to input channel
                                             this) // source processor
      }

      eventChannelArray.add(eventChannel); // eventChannelArray is an OwnedArray, which will
                                           // delete the eventChannel object each time 
                                           // update() is called

   }

Then we will make sure the appropriate variables get re-set at the start of acquisition:

.. code-block:: c++

   void TTLGenerator::startAcquisition()
   {
      counter = 0;
      state = false;
   }

Now, we are ready to add events in our process function:

void TTLEventGenerator::process(AudioSampleBuffer* buffer)
{

   int totalSamples = getNumSamples(0);

   for (int i = 0; i < totalSamples; i++)
   {
      counter++;

      if (counter == sampleRate)
      {

         state = !state;

         uint8 ttlData = state << myChannel;

         TTLEventPtr event = TTLEvent::createTTLEvent(eventChannel, 
                                                      getTimestamp(0) + i, 
                                                      &ttlData, 
                                                      sizeof(uint8), 
                                                      0);

         addEvent(eventChannel, event, i);

         counter = 0;

      }
   }
}

After recompiling the plugin, try dropping it into the signal chain after a :ref:`filereader`. Add an :ref:`lfpviewer` to the right of the plugin, and start acquisition. You should see the state of event channel 1 flipping once per second.

Creating editor class files
#################################

This plugin is going to generate events during acquisition according to settings such as event channel, event interval, manual trigger, etc. An editor interface is required to house these settings. To create such an editor, create two editor class files called :code:`TTLEventGeneratorEditor.cpp` and :code:`TTLEventGeneratorEditor.h`.

After that, open :code:`TTLEventGeneratorEditor.h` and add the following lines of code to the file. The :code:`EditorHeaders.h` file includes all the necessary headers required to create an editor for the plugin. The :code:`TTLEventGeneratorEditor` class inherits members of the `GenericEditor <https://github.com/open-ephys/plugin-GUI/blob/master/Source/Processors/Editors/GenericEditor.h>`__ class. 

.. code-block:: c++

   #include <EditorHeaders.h>
   #include "TTLEventGenerator.h"

   class TTLEventGenerator;

   class TTLEventGeneratorEditor : public GenericEditor
   {
   public:

      TTLEventGeneratorEditor(TTLEventGenerator* parentNode, bool useDefaultParameterEditors);
      ~TTLEventGeneratorEditor();

      void updateSettings() override;

   private:

      TTLEventGenerator* processor;

      JUCE_DECLARE_NON_COPYABLE_WITH_LEAK_DETECTOR(TTLEventGeneratorEditor);

   };

|
| Next, open :code:`TTLEventGeneratorEditor.cpp` and add the following lines of code to the file.

.. code-block:: c++

   #include "TTLEventGenerator.h"
   #include "TTLEventGeneratorEditor.h"

   TTLEventGeneratorEditor::TTLEventGeneratorEditor(TTLEventGenerator* parentNode, bool useDefaultParameterEditors = true)
      : GenericEditor(parentNode, useDefaultParameterEditors)

   {
      processor = parentNode;

      setDesiredWidth(250);
   }

   TTLEventGeneratorEditor::~TTLEventGeneratorEditor(){}

   void TTLEventGeneratorEditor::updateSettings(){}


Now, when you re-compile your plugin and load it into the GUI, it will use this custom editor class. The editor should appear slightly wider than the default, because of the :code:`setDesiredWidth()` command we added.

Adding UI components to the editor
####################################

TODO

Create a button
----------------

TODO - manually trigger an event

Create a slider
----------------

TODO - specify event frequency (0, 5 ms - 5 s)
(50% duty cycle)

Create a combobox
------------------

TODO - select the output channel


Connecting these to parameters in the process method
#####################################################

TODO - use the setParameter() method.


|

