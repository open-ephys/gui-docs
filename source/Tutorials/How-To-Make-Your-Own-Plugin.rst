.. _howtomakeyourownplugin:
.. role:: raw-html-m2r(raw)
   :format: html

How To Make Your Own Plugin
============================

The Open Ephys GUI's plugin architecture allows it to be expanded with external modules (plugins) that can be developed independently of the main application. This is the primary way which we encourage users to add new functionality to the GUI.  

This tutorial will guide you through the steps of making a plugin from scratch by creating a "TTL Event Generator" plugin. :code:`TTL` events are one of three types of events that are supported by the GUI (along with :code:`TEXT` and :code:`BINARY` events). TTL events represent ON/OFF transitions that are traditionally associated with "Transistor-Transitor Logic" circuits. Within the GUI, TTL events are more general, and can also be generated in software. This plugin will add TTL events to a data stream, either at a specified frequency, or whenever a trigger button is pressed. Each TTL channel can change the state of up to 65536 (2^16) "bits," but for simplicity our plugin will only use the first 8 bits.

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


Removing unnecessary files
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

At this point, you should be able to compile your plugin and load it into the GUI. We advise you to compile and test the plugin every time you make changes, so that it is easier for you to identify what changes broke the code, if it happens.

To compile the plugin, please follow the OS-specific instructions as mentioned on the :ref:`compiling plugins <compilingplugins>` page.


Implementing the :code:`process()` method
##########################################

Right now, our plugin doesn't have any effect within the signal chain. Data passed into the :code:`process()` method will not be altered in any way, nor will any events be added to the data stream.

Let's change that by inserting code to add a TTL ON and OFF events at an interval of 1 second. For now, we will hard-code the relevant parameters. In the subsequent steps, we will make it possible to change these parameters via UI elements in the plugin's editor.

Before we can add events during acquisition, we need to announce to downstream processors that this plugin is capable of generating its own events. In the plugin's header file, add the following function declarations, plus some class members:

.. code-block:: c++

   public:
      void createEventChannels() override;

      bool enable() override;

      EventChannel* eventChannel; // pointer to our event channel
   
   private:
      float sampleRate; // holds the sample rate for incoming data

      int counter; // counts the total number of incoming samples
      bool state; // holds the channel state (on or off)

This will allow us to override the default implementation of the :code:`createEventChannels()` method, which is automatically called whenever a plugin needs to update its settings.

Next, in the .cpp file, add the implementation:

.. code-block:: c++

   void TTLEventGenerator::createEventChannels()
   {

      sampleRate = getSampleRate(0);

      const DataChannel* inputChannel = getDataChannel(0);

      if (!inputChannel) // no input channels to this plugin
      {
            eventChannel = new EventChannel(EventChannel::TTL, // channel type
                                             8, // number of bits (up to 65536)
                                             1, // data packet size
                                             sampleRate, // sampleRate
                                             this) // source processor
      } else {
         eventChannel = new EventChannel(EventChannel::TTL, // channel type
                                             8, // number of bits (up to 65536)
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

   bool TTLGenerator::enable()
   {
      counter = 0;
      state = false;

      return isEnabled;
   }

Now, we are ready to add events in our process function:


.. code-block:: c++

   void TTLEventGenerator::process(AudioSampleBuffer* buffer)
   {

      int totalSamples = getNumSamples(0);

      int eventIntervalInSamples = int(sampleRate);

      for (int i = 0; i < totalSamples; i++)
      {
         counter++;

         if (counter == eventIntervalInSamples)
         {

            state = !state;

            uint8 ttlData = state;

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

|
| Then, we need to let the GUI know that the plugin has a custom editor that needs to be created during runtime. To do that, uncomment the following lines of code in the :code:`TTLEventGenerator.h` file...

.. code-block:: c++

   bool hasEditor() const {return true;}

   AudioProcessorEditor* createEditor() override;


...and add the following lines of code to the :code:`TTLEventGenerator.h` file

.. code-block:: c++

   AudioProcessorEditor* TTLEventGenerator::createEditor()
   {
      editor = new TTLEventGeneratorEditor(this, true);
      return editor;
   }

Now, when you re-compile your plugin and load it into the GUI, it will use this custom editor class. The editor should appear slightly wider than the default, because of the :code:`setDesiredWidth()` command we added.

Adding UI components to the editor
####################################

Now that our plugin is able to generate events in the process() method and has its own editor, lets add some UI components to the editor that allows the user to change various parameters of the plugin.

Create a button
----------------

To allow triggering events manually, let's add the button to the editor that the user can click on to generate an event. First of all, add the following class members to the editor's header file:

.. code-block:: c++
   
   private:
      ScopedPointer<UtilityButton> manualTrigger;

:code:`manualTrigger` is a ScopedPointer to a UtilityButton, a type of button that's part of the Open Ephys Plugin API. Because this is a ScopedPointer, it will automatically get deleted when the plugin is removed from the signal chain.

Then, in the plugin editor's we'll initialize the button, add a button listener, set the bounds, and make it visible in the editor by adding the following lines of code to the :code:`TTLEventGeneratorEditor()` constructor:

.. code-block:: c++

   manualTrigger = new UtilityButton("Trigger", Font("Default", 20, Font::plain)); // button text, font to use
   manualTrigger->addListener(this); // add listener to the button
   manualTrigger->setBounds(130, 35, 75, 25); // (x, y, width, height)
   addAndMakeVisible(manualTrigger);  // add the button to the editor and make it visible

We also need to handle button clicks by implementing the :code:`buttonEvent` method as below. For now, keep it empty, we'll come back to it later. Compile and load the plugin into the GUI to see the newly added button.

.. code-block:: c++

   void TTLEventGeneratorEditor::buttonEvent(Button* button)
   {
      if(button == manualTrigger)
      {
         // Do something
      }
   }

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-03.png
  :alt: Create a button

Create a slider
----------------

To automatically generate events at certain intervals/frequency, lets add a slider with a range of event frequency between 5 ms to 5000 ms. Add the following function declaration and class member to the editor's header file:

.. code-block:: c++

   public:
      void sliderEvent(Slider* slider) override; // handle slider change events

   private:
      ScopedPointer<Slider> eventFrequency;
      ScopedPointer<Label> frequencyLabel;

Then, we'll initialize the slider, set range, set bounds, add a listener, add a label and make the slider and label visible by adding the following lines of code to the :code:`TTLEventGeneratorEditor()` constructor:

.. code-block:: c++

	eventFrequency = new Slider();
	eventFrequency->setRange(50, 5000, 50);  // range between 50 ms to 5000 ms with 50 ms intervals
	eventFrequency->setValue(50);
	eventFrequency->setTextValueSuffix (" ms");
	eventFrequency->addListener (this);
	eventFrequency->setChangeNotificationOnlyOnRelease(true);
	eventFrequency->setBounds(25, 95, 200, 25);
	addAndMakeVisible (eventFrequency);

   frequencyLabel = new Label("FrequencyLabel", "Frequency");
   frequencyLabel->attachToComponent(eventFrequency, false);
   addAndMakeVisible(frequencyLabel);

We also need to handle slider value changes by implementing the :code:`sliderEvent` method. For now, keep it empty, we'll come back to it later. Compile and load the plugin into the GUI to see the newly added slider.

.. code-block:: c++

   void TTLEventGeneratorEditor::sliderEvent(Slider* slider)
   {
      if(slider == eventFrequency)
      {
         // Do something
      }
   }

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-04.png
  :alt: Create a slider

Create a ComboBox
------------------

To select which channel to generate and show events in, a ComboBox needs to be created. This will allow the user to select a TTL output bit from a drop-down menu. First, add :code:`ComboBox::Listener` as a base class to :code:`TTLEventGeneratorEditor` as that is not added by GenericEditor unlike button and slider listener classes. Then, add the following function declaration and class members to the editor's header file:

.. code-block:: c++
   
   public:
      void comboBoxChanged(ComboBox* comboBox) override;

   private:
      ScopedPointer<ComboBox> outputBitSelector;
	   ScopedPointer<Label> outputLabel;

Then, initialize the ComboBox, add output channels to the list, set bounds, add listener, add label and make it visible by adding the following lines of code to the :code:`TTLEventGeneratorEditor()` constructor:

.. code-block:: c++

   outputBitSelector = new ComboBox();

   for (int bit = 1; bit <= 8; bit++)
      outputBitSelector->addItem(String(bit), bit);

   outputBitSelector->setSelectedId(1);
   outputBitSelector->setBounds(50,35,50,25);
   outputBitSelector->setTooltip("Output event channel");
   outputBitSelector->addListener(this);
   addAndMakeVisible(outputBitSelector);

   outputLabel = new Label("Bit Label", "OUT");
   outputLabel->attachToComponent(outputBitSelector, true);
   addAndMakeVisible(outputLabel);

Just like the button and slider, We also need to handle combobox value changes by implementing the :code:`comboBoxChanged` method. For now, keep it empty, we'll come back to it later. Compile and load the plugin into the GUI to see the newly added ComboBox.

.. code-block:: c++

   void TTLEventGeneratorEditor::comboBoxChanged(ComboBox* comboBox)
   {
      if(comboBox == outputBitSelector)
      {
         // Do something
      }
   }

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-05.png
  :alt: Create a combobox

Connecting these to parameters in the :code:`process()` method
###############################################################

Now, let's allow our UI elements to change the state of the plugin. To do this, we need to create variables inside the :code:`TTLEventGenerator` class that can be updated by our button, slider, and ComboBox. The values of these variables *must* be updated through a special method, called :code:`setParameter()`, which takes two inputs, a parameter ID and a value. This is because the :code:`process()` method is called by a separate thread from the user interface, and the variables it needs to access can only be updated at specific times. Modifying variables via :code:`setParameter()` ensures that they are handled properly, and prevents unexpected behavior or segmentation faults.

First, let's uncomment the :code:`setParameter()` method and then add three variables to the :code:`TTLEventGenerator` header file to store the state of our three parameters:

.. code-block:: c++

   public:
      void setParameter(int parameterIndex, float newValue) override;

   private:
      bool shouldTriggerEvent;
      bool eventWasTriggered;
      int triggeredEventCounter;

      float eventIntervalMs;
      int outputBit;
   
Next, let's initialize their values in the :code:`TTLEventGenerator()` constructor method.

.. code-block:: c++

   shouldTriggerEvent = false;
   eventWasTriggered = false;
   triggeredEventCounter = 0;

   eventIntervalMs = 50.0f;
   outputBit = 0; // the GUI uses 0-based indexing internally, even though the 
                  // user-facing labels use 1-based indexing

.. important:: Always be sure to initialize all member variables, in order to avoid unexpected behavior.

Now, we can define how these variables are updated inside the :code:`setParameter()` method:

.. code-block:: c++

   void TTLEventGenerator::setParameter(int ID, float value)
   {

      if (ID == 0)
      {
         shouldTriggerEvent = true;
      } else if (ID == 1)
      {
         eventIntervalMs = value;
      } else if (ID == 2)
      {
         outputBit = int(value);
      }
   }


Next, we'll edit the :code:`buttonEvent`, :code:`sliderEvent`, and :code:`comboBoxChanged` classes to call to :code:`setParameter()`:

.. code-block:: c++

   void TTLEventGeneratorEditor::buttonEvent(Button* button)
   {
      if(button == manualTrigger)
      {
         setParameter(0, 0.0f); // the second input is arbitrary in this case
      }
   }

.. code-block:: c++

   void TTLEventGeneratorEditor::sliderEvent(Slider* slider)
   {
      if(slider == eventFrequency)
      {
         setParameter(1, slider->getValue());
      }
   }

.. code-block:: c++

   void TTLEventGeneratorEditor::comboBoxChanged(ComboBox* comboBox)
   {
      if(comboBox == outputBitSelector)
      {
         setParameter(2, comboBox->getSelectedId() - 1); // subtract 1 to convert to zero-based indexing
      }
   }

Finally, we need to update our process method to make use of these parameters:


.. code-block:: c++

   void TTLEventGenerator::process(AudioSampleBuffer* buffer)
   {

      int totalSamples = getNumSamples(0);

      int eventIntervalInSamples = (int) sampleRate * eventIntervalMs / 2 / 1000;

      if (shouldTriggerEvent)
      {

         // add an event at the first sample.
         uint8 ttlData = true << outputBit;

         TTLEventPtr event = TTLEvent::createTTLEvent(eventChannel, 
                                                         getTimestamp(0), 
                                                         &ttlData, 
                                                         sizeof(uint8), 
                                                         outputBit);

         addEvent(eventChannel, event, 0);

         shouldTriggerEvent = false;
         eventWasTriggered = true;
         triggeredEventCounter = 0;
      }

      for (int i = 0; i < totalSamples; i++)
      {
         counter++;

         if (eventWasTriggered)
            triggeredEventCounter++;

         // Turn off manually triggered event after selected interval
         if (triggeredEventCounter == eventIntervalInSamples)
         {
            uint8 ttlData = 0;

            TTLEventPtr event = TTLEvent::createTTLEvent(eventChannel, 
                                                         getTimestamp(0) + i, 
                                                         &ttlData, 
                                                         sizeof(uint8), 
                                                         outputBit);

            addEvent(eventChannel, event, i);

            eventWasTriggered = false;
            triggeredEventCounter = 0;
         }

         if (counter == eventIntervalInSamples)
         {

            state = !state;

            uint8 ttlData = state << outputBit;

            TTLEventPtr event = TTLEvent::createTTLEvent(eventChannel, 
                                                         getTimestamp(0) + i, 
                                                         &ttlData, 
                                                         sizeof(uint8), 
                                                         outputBit);

            addEvent(eventChannel, event, i);

            counter = 0;

         }

         // reset counter to 0 if it goes out of bounds 
         if (counter > eventIntervalInSamples)
            counter = 0;
      }
   }

And that's it! If you compile and test your plugin, the UI elements in the editor should now change the events that appear in the LFP Viewer.

.. image:: ../_static/images/tutorials/makeyourownplugin/makeyourownplugin-05.png
  :alt: Plugin in signal chain

Next steps
#############

This plugin are a number of ways this plugin could be enhanced. To practice creating different kinds of UI elements, you could try implementing some of the features below, or come up with your own!

- Ensure an "OFF" event is sent when the output bit is changed.

- Add a button that turns the plugin's output on and off.

- Add an editable label that can be used to define the time between ON/OFF events (currently the output bit flips at a 50% duty cycle).

|

