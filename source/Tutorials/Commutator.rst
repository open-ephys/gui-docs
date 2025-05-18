.. _commutator:

..  role:: raw-html-m2r(raw)
    :format: html

Automated Tether Commutation
============================

..  note::
    Following this tutorial requires a 3D capable headstage, an Open Ephys SPI commutator, and a 3D capable data acquisition system (e.g. a Gen 3 Acquisition Board or an ONIX board).

Most acquisition systems rely on a tether to transmit power and data between the headstage and the
data acquisition system. This can be problematic during freely behaving experiments because the
animal is liable to twist and tangle the tether while it moves around which can end up exerting
torque on the animal. This is mitigated by using a commutator, a device that untwists the tether as
the animal moves around while maintaining electrical continuity between the animal and data
acquisition system. This encourage more naturalistic behaviors.

This tutorial demonstrates how to automate commutation using a the Open Ephys GUI, a 3D-capable
headstage, and an Open Ephys SPI commutator. 

Hardware connections
#####################

..  tab-set::
    :sync-group: acquisition-hardware

    ..  tab-item:: Acquisition Board
        :sync: acquisition-board

        #.  Follow the `Acquisition Board Quick Start Guide
            <https://open-ephys.github.io/acq-board-docs/User-Manual/Quickstart-guide.html>`__ to establish
            the following acquisition board connections:

            -   USB 3.0 connection between the acquisition board and the PC.

            -   +5V connection between the acquisition board and an AC power source.

        #.  Follow the `SPI Commutator Connections section
            <https://open-ephys.github.io/commutator-docs/user-guide/mount-connect.html?commutator=spi#connecting>`__
            of the commutator hardware docs to establish the following commutator connections:

            -   SPI connection between the commutator's stator and the acquisition board.

            -   SPI connection between the commutator's rotor and the 3D capable headstage.
            
            -   USB connection between the commutator and the PC.   

    ..  tab-item:: ONIX
        :sync: onix

        #.  Follow the `ONIX Hardware Guide
            <https://open-ephys.github.io/onix-docs/User-Manual/Quickstart-guide.html>`__ to setup
            your ONIX board and establish the following ONIX system's connections:

            -   2x (A & B) coaxial connections between the breakout board and the PCIe host.

            -   SDR connection between the breakout board and the PCIe host.

        #.  Follow the `Coax Commutator Connections section
            <https://open-ephys.github.io/commutator-docs/user-guide/mount-connect.html?commutator=coax#connecting>`__
            of the commutator hardware docs to establish the connections between the ONIX system and a 3D capable headstage:

            -   Coaxial connection between the commutator's stator and the acquisition board.

            -   Coaxial connection between the commutator's rotor and the 3D capable headstage.

            -   USB connection between the commutator and the PC.   


Install GUI and prepare signal chain
####################################

#.  :doc:`Install the Open Ephys GUI </User-Manual/Installing-the-GUI>` if you haven't already.

#.  Open the GUI. 

    ..  note::
        -   If this is your first time opening the GUI, you will be queried to load a default signal chain. Choose the "Acquisition Board" signal chain.
        -   If the Open Ephys GUI already has a signal chain loaded you would like to save, :ref:`do that <file>` before proceeding to the next step because the next step overwrites any current signal chain.

#.  Download one following signal chains depending on which hardware you are using.

    ..  tab-set::
        :sync-group: acquisition-hardware

        ..  tab-item:: Acquisition Board
            :sync: acquisition-board

            :download:`Acquisition Board Signal Chain`

            ..  image:: /_static/images/tutorials/commutator/onix-signal-chain.png
                :alt: Acquisition Board Signal Chain for commutation

        ..  tab-item:: ONIX
            :sync: onix

            :download:`ONIX Signal Chain`

            ..  image:: /_static/images/tutorials/commutator/onix-signal-chain.png
                :alt: ONIX Signal Chain for commutation

#.  :ref:`Open <file>` the downloaded signal chain in the GUI.

#.  Refer to :doc:`/User-Manual/Plugins/Commutator-Control` to configure the Commutator Control
    processor.

    -   The selected stream should correspond to a port that has a 3D capable headstage to it.

    -   The selected COM port should correspond to the one to which your commutator is connected. 

    -   For an off-the-shelf Open Ephys 3D capable headstage, adjusting the rotation axis is not
        necessary.

#.  Click the ▶ play button in the top-right corner of the GUI. The commutator now follows the
    rotation of the headstage. 

