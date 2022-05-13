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

Text goes here.

Visualizer class methods
#########################

.. function:: void resized() override

   Updates boundaries of sub-components whenever the canvas size changes

.. function:: void refreshState()

   Called when the visualizer's tab becomes visible again 

.. function:: void update()

   Updates settings 

.. function:: void refresh()

   Called instead of "repaint()" to avoid re-painting sub-components

.. function:: void paint(Graphics& g)

   Draws the canvas background

Visualizer editor methods
#########################

Constructor
------------



Saving/loading settings
########################

Text goes here.

Interactive Plot
########################

Visualizers that display data can take advantage of the GUI's :code:`InteractivePlot` class to draw 2D charts.

If a 

.. function:: void plot(std::vector<float> x, 
			  std::vector<float> y, 
			  Colour c = Colours::white,
			  float width = 1.0f,
			  float opacity = 1.0f,
			  PlotType type = PlotType::LINE)

   Adds a line based on X and Y values

.. function:: void clear()

   Clears all lines from the plot 


.. function:: void show()

   Draws the plot


.. function:: void title(String t)

   Adds a title to the plot


.. function:: void xlabel(String label)

   Sets the x-axis label
	

.. function:: void ylabel(String label)

   Sets the y-axis label


.. function:: void setInteractive(InteractivePlotMode mode)

   Set whether the plot is interactive


.. function:: void showXAxis(bool state)

   Sets whether x-axis is visible


.. function:: void showYAxis(bool state)

   Sets whether y-axis is visible


.. function:: void showGrid(bool state)

   Sets whether grid is visible


.. function:: void setBackgroundColour(Colour c)

   Sets the background colour


.. function:: void setGridColour(Colour c)

   Sets the colour of the grid.

.. function:: void setAxisColour(Colour c)

   Sets the colour of the axes. 


.. function:: void setRange(XYRange& range)

   Sets range of both axes


.. function:: void getRange(XYRange& range)

   Copies the current range values