.. _visualizerplugins:

.. default-domain:: cpp

Visualizer Plugins
=====================

.. csv-table:: Visualizers are Processor Plugins that include a canvas for displaying data.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::PROCESSOR`"
   "*Base Classes*", ":code:`GenericProcessor`, :code:`VisualizerEditor`, :code:`Visualizer`"
   "*Template*", "https://github.com/open-ephys-plugins/visualizer-plugin-template"

Overview
#####################

All plugins for the Open Ephys GUI have space for parameter editors and small visualizations inside of their "editor," which lives inside the Editor Viewport along the bottom of the main GUI window. Because there is a limited amount of screen real estate inside the editor, plugin developers can choose to display additional information inside a "canvas" that lives inside the GUI's Viewport, or optionally inside a separate window. This makes it possible to create rich visualizations of multichannel data (such as the :ref:`lfpviewer` and :ref:`spikeviewer`), or to provide a more comprehensive interface for adjusting parameters (such as in the :ref:`neuropixelspxi` or :ref:`crossingdetector` plugins).

Any :ref:`processorplugins` or :ref:`datathreads` can become Visualizer plugins simply by deriving their editors from the :code:`VisualizerEditor` class, rather than the :code:`GenericEditor` class. A Visualizer Editor contains two buttons in the upper right that specify whether the Visualizer canvas should appear in a tab inside the GUI's Viewport or in a separate window. The Visualizer Editor will automatically track the state of these buttons, and will ensure that the Visualizer is re-loaded in the same location each time.

Visualizer Editor methods
#########################

The Visualizer Editor constructor has two more parameters than the standard Generic Editor constructor:

.. function:: VisualizerEditor(GenericProcessor* processor, String tabText, int desiredWidth)

   Constructor for a Visualizer Editor.

   :param processor: Pointer to the :code:`GenericProcessor` associated with this editor.
   :param tabText: The text that will appear in the Visualizer tab within the GUI's Viewport (or the title of the Visualizer window).
   :param desiredWidth: The width (in pixels) of this editor. If the editor needs to adjust its size at a later time, it simply needs to change the value of its :code:`desiredWidth` member.

A Visualizer Editor must also implement the following method to specify how its Visualizer should be created:

.. function:: Visualizer* createNewCanvas()

   :returns: A pointer to a :code:`Visualizer` object.


The following methods are needed to start and stop Visualizer animation callbacks. This is usually done when starting and stopping acquisition, but they can also be called at other times as needed.

.. function:: void enable()

   Calls Visualizer's :code:`beginAnimation()` method, which starts the animation timer.

.. function:: void disable()

   Calls Visualizer's :code:`endAnimation()` method, which stops the animation timer.


Visualizer methods
#########################

A Visualizer is simply a Juce :code:`Component` that includes an animation timer. Anything that can be done inside a :code:`Component` can be done inside a Visualizer.

All Visualizers must implement three pure virtual methods:

.. function:: void refresh()

   Called on each animation cycle; the Visualizer should make sure all the relevant components are re-drawn inside this method. This is used iinstead of Juce's :code:`repaint()` to avoid re-painting sub-components that don't need to be updated. To modify the animation refresh rate, a Visualizer should change the value of its :code:`refreshRate` member.


.. function:: void refreshState()

   Called once when the Visualizer's tab becomes visible.


.. function:: void update()

   Called when the signal chain is modified, to allow the Visualizer to update its internal settings. Note that Visualizer settings will not be propagated to downstream plugins.


In addition, a Visualizer should override the following Juce :code:`Component` methods to specify its background and layout:

.. function:: void paint(Graphics& g)

   Draws the Visualizer background.

.. function:: void resized()

   Updates boundaries of sub-components whenever the size of the Visualizer is changed.


Saving/loading settings
########################

If a Visualizer Editor contains parameter editors that do not inherit from the GUI's built in :code:`ParameterEditor` class, it must save and load its settings by overriding the following methods (note that these differ from what is used by the :code:`GenericEditor` class):

.. function:: void saveVisualizerEditorParameters(XmlElement* xml)

   Saves any custom parameters within a Visualizer Editor. Parameters should be added as attributes of the :code:`XmlElement` that's passed into this method.


.. function:: void loadVisualizerEditorParameters(XmlElement* xml)

   Loads any custom parameters within a Visualizer Editor. Parameters are stored as attributes of the :code:`XmlElement` that's passed into this method.

To save and load custom parameters within the Visualizer itself, the following methods should be overridden:

.. function:: void Visualizer::saveCustomParametersToXml(XmlElement* xml)

   Saves any custom parameters within a Visualizer. Parameters should be added as attributes of the :code:`XmlElement` that's passed into this method.


.. function:: void Visualizer::loadCustomParametersFromXml(XmlElement* xml)

   Loads any custom parameters within a Visualizer. Parameters are stored as attributes of the :code:`XmlElement` that's passed into this method.


Interactive Plots
########################

Visualizers that display data can take advantage of the GUI's :code:`InteractivePlot` class to draw 2D charts.

The following methods define the behavior of an Interactive Plot:

.. function:: void plot(std::vector<float> x, std::vector<float> y, Colour c = Colours::white, float width = 1.0f, float opacity = 1.0f, PlotType type = PlotType::LINE)

   Adds a plot element based on a vector of X and Y values.

   :param x: A vector of locations along the X-axis.
   :param y: A vector of locations along the Y-axis.
   :param c: The color of the plot element.
   :param width: The width of the line (in the case of a line plot), the width of the dots (in the case of a scatter plot), or the width of the bars (in the case of a bar plot).
   :param opacity: The opacity of the plot element.
   :param type: The type of plot element to draw (options are :code:`LINE`, :code:`SCATTER`, :code:`BAR`, and :code:`FILLED`)


.. function:: void clear()

   Clears all elements from the plot.


.. function:: void show()

   Draws all elements that have been added since the plot was cleared.


.. function:: void title(String t)

   Adds a title to the plot.

   :param t: The title text.


.. function:: void xlabel(String label)

   Sets the x-axis label.

   :param label: The x-axis label text.
	

.. function:: void ylabel(String label)

   :param label: The y-axis label text.


.. function:: void setInteractive(InteractivePlotMode mode)

   Set whether the plot can be panned and zoomed.

   :param mode: Can be either :code:`ON` or :code:`OFF`


.. function:: void showXAxis(bool state)

   Sets whether x-axis is visible.

   :param state: :code:`true` if the x-axis should be drawn.


.. function:: void showYAxis(bool state)

   Sets whether y-axis is visible.

   :param state: :code:`true` if the y-axis should be drawn.


.. function:: void showGrid(bool state)

   Sets whether plot grid is visible.

   :param state: :code:`true` if the grid should be drawn.


.. function:: void setBackgroundColour(Colour c)

   Sets the background colour of the plot.

   :param c: The background colour.


.. function:: void setGridColour(Colour c)

   Sets the colour of the plot grid.

   :param c: The grid colour.


.. function:: void setAxisColour(Colour c)

   Sets the colour of the axes. 

   :param c: The axis colour.


.. function:: void setRange(XYRange& range)

   Sets the range of both axes.

   :param range: An :code:`XYRange` object consisting of four values (x-min, x-max, y-min, and y-max).


.. function:: void getRange(XYRange& range)

   Copies the current range values.

   :param range: An :code:`XYRange` object consisting of four values (x-min, x-max, y-min, and y-max).
