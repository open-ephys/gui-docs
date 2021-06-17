.. _faq:
.. role:: raw-html-m2r(raw)
   :format: html

FAQs
============================


**I'd like to use the Open Ephys GUI, but I'm worried that I won't be able to get help when I need it. What support options are available?**

In many cases, answers to common inquiries can be found by searching through our `forum <https://groups.google.com/g/open-ephys>`__ or `GitHub Issues page <https://github.com/open-ephys/plugin-GUI/issues>`__. If you can't find what you're looking for, we encourage you to post a question yourself.

If you'd prefer to reach us by email, simply write to :code:`support@open-ephys.org` and we'll be happy to help. Support efforts are funded by sales from the `Open Ephys store <https://open-ephys.org/store>`__, as well as a `BRAIN Initiative U24 award <https://reporter.nih.gov/project-details/9645567>`__ to the Allen Institute.

|

**Is the GUI robust enough to carry out serious scientific research?**

Absolutely. The GUI has been used to collect data for `over 200 publications and preprints <https://open-ephys.org/publications>`__ , which have appeared in every major neuroscience journal. 

Crashes are still possible, but they usually happen when testing out new features or new combinations of plugins. Once you find a signal chain that works for your needs, it can be used for many hours of uninterrupted recording.

|

**My current data acquisition software does X. Can you add this feature to the Open Ephys GUI?**

To request a new feature, please `post an issue on GitHub <https://github.com/open-ephys/plugin-GUI/issues>`__. If it seems like something that would benefit a reasonably large fraction of the Open Ephys user base, we'll consider adding it to our `development roadmap <https://github.com/open-ephys/plugin-GUI/issues/435>`__. As with nearly all neuroscience-specific software projects, our development resources are extremely limited. But we love to learn about the needs of the community, so don't hesitate to let us know what you'd like to see in a future release!

|

**How do I synchronize Open Ephys with other software?**

Synchronization is one of the most important issues we face in our experiments – if you don't get it right, it can make it much more difficult to analyze your data, or even make it completely uninterpretable.

If you're using multiple applications to acquire data in parallel, there are two general approaches to synchronization:

(1) *Software-based triggering.* The GUI's :ref:`networkevents` plugin can be used to start your recordings from a Python or Matlab script. Assuming your other software has similar remote triggering functionality, this will ensure that all of your data streams begin within a few milliseconds of one another.

(2) *Shared hardware sync line.* For precise, sub-millisecond level synchronization, all of your data streams need to share at least one digital or analog input line. This line should includes a sequence of on/off events that can be used to align the data offline. To learn more about hardware-based synchronization, check out THIS TUTORIAL.

For synchronizing the GUI with camera streams, you can either record the frame triggers (if available) or record a digital input that activates an LED in the camera's field of view. We recommend using `Bonsai <https://bonsai-rx.org/>`__ for acquiring video data.

|

**Can I use the GUI for human EEG experiments?**

Currently, none of the GUI's compatible hardware interfaces are optimized for scalp EEG recording. It is possible to `adapt the Open Ephys acquisition board for use with EEG electrodes <https://iopscience.iop.org/article/10.1088/1741-2552/aa651f>`__, but this is not an officially supported approach.

If you're interested in performing human EEG experiments, we recommend checking out `OpenBCI <https://openbci.com/>`__.

|

**I love using open-source tools, but I'm not a developer myself. How can I contribute?**

First and foremost, simply using the GUI is extremely helpful. Feedback about bugs and feature requests helps guide our development process and makes the software better for everyone. If you subscribe to our `forum <https://groups.google.com/g/open-ephys>`__, we'll send out requests to help test out new features prior to each release.

If you know someone with software engineering skills, you can advocate for them to be able to spend a portion of their time working on the GUI. We are more than happy to suggest projects and supervise developers along the way.

If you want to contribute in a more substantial way, you could add funding for a software developer to your next grant application. We will gladly provide a letter of support, or even contribute to the writing process. Get in touch with :code:`info@open-ephys.org` if this is something you're considering.

|

**Can I license the Open Ephys GUI for a commercial project?**

The GUI is released under a `GPL 3.0 license <https://github.com/open-ephys/plugin-GUI/blob/master/LICENSE>`__, which means that any derivative software much also be open source. However, if you'd like to use the GUI to acquire data from a closed-source device, this is both easy and encouraged! We would love for more companies to use the GUI, rather than choosing to build their own acquisition software from scratch. 

|