.. _matlabinterface:
.. role:: raw-html-m2r(raw)
   :format: html

################
MATLAB Interface
################

.. image:: ../../_static/images/plugins/matlabinterface/matlabinterface-01.png
  :alt: Annotated Matlab Interface editor

.. csv-table:: Streams multiple channels of continuous data from the Open Ephys GUI to a live Matlab session. A Matlab API allows seamless processing of the incoming data in real time.
   :widths: 18, 80

   "*Plugin Type*", "Filter"
   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Pavel Kulik"
   "*Source Code*", "https://github.com/open-ephys-plugins/matlab-interface"

Installing and upgrading
###########################

..
   The Matlab Interface plugin is not included by default in the Open Ephys GUI. To install, use **ctrl-P** or **⌘P** to open the Plugin Installer, browse to the "Matlab Interface" plugin, and click the "Install" button.

   The Plugin Installer also allows you to upgrade to the latest version of this plugin, if it's already installed.

The Matlab Interface is not yet released for version 1.0 of the Open Ephys GUI. An announcement will be made via the Open Ephys Discord when it's available for download via Plugin Installer.

Plugin configuration
########################

The plugin will automatically open a socket using the IP and Port address listed in the Matlab Interface editor.

To initialize the connection on the Matlab side, you will need to call your generated script from Matlab following the instructions below. Once the connection has been initiated on both sides, pressing the "Play" button in the Open Ephys GUI will automatically stream the incoming data to Matlab, and then back to the GUI for further processing. 

Matlab API
############

In order to process the incoming data in Matlab, you will need to leverage the included Matlab API.

The Matlab API is centered around a :code:`GenericProcessor` class that encapsulates an Open Ephys data processor. The idea is to write your own class that inherits and extends :code:`GenericProcessor`. A starting template is included below: 

.. code-block:: matlab

   classdef MyClass < GenericProcessor

	    properties
	        %define any variables you want to keep track of here (see examples)
	    end

	    methods 
	        function self = MyClass(host, port)
	            self = self@GenericProcessor(host, port);
	            %Initialize any variables here (see examples)
	            self.process();
	        end
	    end

	    methods (Access = protected)
	        function process(self)
	            while (true) 
	                process@GenericProcessor(self); 
	                numSamples = self.dataIn.numSamplesFetched;
	                data = self.dataIn.continuous(1:end);
	                %Do whatever you want with the data here (see examples)
	            end
	        end
	    end
	end

Additional examples can be found in the plugin's `README file <https://github.com/open-ephys-plugins/matlab-interface/blob/main/README.md>`__.

The following screenshot shows data being transmitted from the Open Ephys GUI and plotted in real time in Matlab:

.. image:: ../../_static/images/plugins/matlabinterface/matlab_interface_screenshot.png
  :alt: Data streaming into Matlab