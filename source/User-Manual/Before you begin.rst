.. _beforeyoubegin:
.. role:: raw-html-m2r(raw)
   :format: html

Before you begin
=====================

|

If this is your first time learning about the Open Ephys GUI, welcome! This software was designed with community in mind, and we hope you will become a part of ours. We want scientists to take ownership of their tools, in order to ensure that the latest experimental techniques are available to everyone, no matter their level of funding or expertise. But this ideal won't be reached until we have buy-in from a much larger portion of neuroscientists.

**Why did we build the GUI?** At a high level, we believe that progress is hindered when scientists have to choose between closed-source, mutually incompatible data acquisition platforms. As an analogy, imagine if every lab had to pick only one of ten different journals to disseminate all of their results, and they were only allowed to read articles that were published in their chosen journal. Clearly, this would lead to fragmentation across the field, with ideas spreading less freely than if new findings were available to all. This is what is currently happening with our experimental methods, where for many scientists, the latest techniques are inacessible unless a company decides it is profitable to sell them.

We don't think that the Open Ephys GUI is the perfect solution to this problem, but it is a step in the right direction. If these ideas appeals to you, we encourage you to take the time to learn about what we stand for, and whether the GUI is right for you.

About Open Ephys
-----------------

`Open Ephys`_ is a US-based nonprofit founded by Josh Siegle and Jakob Voigts in order to promote the use of open-source tools for electrophysiology experiments. Open Ephys showcases open-source tools developed in academia, some of which are sold under the "Open Ephys" brand, while others are independent. Through a partnership with `Open Ephys Production Site`_ in Portugal, we are able to sell open-source hardware that is validated, tested, and supported, all at an affordable price. Purchases made through the `Open Ephys Store`_ ensure that your money is invested back into the community.

The `Open Ephys GUI`_ is often just called "Open Ephys," but it's actually part of a wider range of software and hardware that is part of the Open Ephys ecosystem. Development of the GUI began in the Wilson Lab at MIT around 2010. In 2014, Aarón Cuevas López was hired as the official support person, using revenues from the `Open Ephys Store`_. Since 2019, support for the GUI has been funded by a `BRAIN Initiative U24 Award`_ to the Allen Institute.

The philosopy behind the GUI
----------------------------

When we started building the Open Ephys GUI, there were many existing options for acquiring extracellular electrophysiology data. But none of them met all three of the following criteria:

* **Open source.** We consider this the most essential feature of scientific software. Performing cutting-edge neuroscience experiments requires a degree of flexibility that just isn't possible when you have to rely on a company to implement the features you need. In order to stay ahead of the curve, scientists must be able to peer underneath the hood of their tools, understand their behavior, and modify them if necessary.

* **Plugin based.** We quickly realized that being "open source" was not enough to foster community engagement. In order to convince others to contribute to a piece of software, there needs to be an easy way to extend its functionality. Plugin/host architectures are widely used, and ensure that developers don't need to understand the entire software to extend its capabilities. Plus, plugins can be distributed and upgraded separately from the main application, making it straightforward for scientists to share their innovations.

* **Cross platform.** Many labs already rely on software that is restricted to a particular operating system, so we wanted to ensure that the GUI would be compatible with any existing setup.

Until recently, the GUI was the only software designed for extracellular electrophysiology experiments that met these three criteria. However, when Microsoft released .NET as a cross-platform framework, `Bonsai`_ (another open-source, plugin-based application) was no longer restricted to the Windows operating system. We think `Bonsai`_ is the future of neuroscience data acquisition, but it's currently more of a general-purpose visual programming language that lacks many of the convenient visualizations built into the Open Ephys GUI. We highly recommend learning more about `Bonsai`_, because it may be better suited for the types of experiments you'd like to do. It can also be used in combination with the Open Ephys GUI to facilitate more complex closed-loop manipulations (see `this paper`_ for an example of how this can work in practice).

Strengths and weaknesses
----------------------------------

While you shouldn't form an opinion about the GUI until you've tested it yourself, reading this section will give you a better idea about what to expect. The following are three key aspects of the GUI that have both positive and negative impacts on the overall user experience.

* **It's built with flexibility in mind.** If you open up the GUI and immediately press "play," nothing will happen. That's because the software doesn't dictate in advance what your signal chain should be. Instead, you need to define your own data processing stream, using a combination of plugins that suits your experimental needs. This allows the GUI to be used for a variety of different applications, but it also means some learning is necessary before you'll know how to use it properly. There's also a tradeoff between flexibility and stability, as it's hard to guarantee that every possible combination of plugins will work as expected. The GUI has been used to reliably collect countless terabytes of high-quality electrophysiology data, but don't be surprised if you encounter bugs and crashes when you're testing out new features.

* **It was developed by scientists, for scientists.** The GUI grew out of the general frustration that existing software for electrophysiology lacked the elegance of applications in other domains, such as music production. We set out to create something that was both powerful and fun, which we didn't mind using on a daily basis. But designing your own software is a double-edged sword. While the GUI has made it straightforward to carry out experiments that would be difficult to implement in any other way, there is a big difference between getting something to work once, and designing something performs well under a wide array of use case. As you familiarize yourself with the GUI or its underlying source code, you'll find some features to be incredibly intuitive, and while others make you question how anyone could possibly think this was a good idea. Fortunately, everything is out in the open, so if something isn't working the way you expect it to, you are free to change it yourself or ask someone else for help.

* **The backend uses components originally designed to handle audio data.** Neural signals have a remarkably similar bandwidth to audible sound, so it made sense to take advantage of a powerful audio library (called `JUCE`_) to handle the complexities of real-time data processing. Continuous signals are passed around via :code:`AudioBuffers`, while spikes and other events are packaged into :code:`MidiBuffers`. The fact that `JUCE`_'s audio classes could be easily adapted to process neural data made the GUI practical to build. But it also meant we had to implement some hacks to handle data streams with a variety of sampling rates. If you'd like to carry out experiments that require real-time integration of many asynchronous signals, `Bonsai`_ may be a better choice.

Things to keep in mind
----------------------

The Open Ephys GUI is freely available to anyone who wants to use it. However, we have a few requests of our users, which should also apply to any open-source software.

* **Test any features before you use in them in your experiments.**  The GUI is not guaranteed to work as advertised. Whenever you download or upgrade the GUI, be sure to test your desired configuration in a "safe" environment before using it to collect real data.

* **Your input is essential for making the GUI better.** If you observe any unexpected behavior, please `report an issue`_ as soon as possible. We rely on help from the community to ensure that the GUI is functioning properly.

* **Be sure to include it in your methods section.** Any publications based on data collected with the GUI should include a reference to `Siegle et al. (2017)`_, as citations remain essential for measuring the impact of scientific software.

|
|

.. _Open Ephys: https://open-ephys.org
.. _Open Ephys Production Site: https://www.oeps.tech/
.. _Open Ephys Store: https://open-ephys.org/store
.. _Open Ephys GUI: https://open-ephys.org/gui
.. _BRAIN Initiative U24 Award: https://projectreporter.nih.gov/project_info_description.cfm?aid=9645567
.. _Bonsai: https://bonsai-rx.org/
.. _this paper: https://iopscience.iop.org/article/10.1088/1741-2552/aacf45/meta
.. _JUCE: https://juce.com/
.. _report an issue: https://github.com/open-ephys/plugin-GUI/issues
.. _Siegle et al. (2017): https://iopscience.iop.org/article/10.1088/1741-2552/aa5eea