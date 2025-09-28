.. _commutator:

..  role:: raw-html-m2r(raw)
    :format: html

Automating Tether Commutation
==============================

..  note::
    Following this tutorial requires a 3D capable headstage, an Open Ephys SPI 
    commutator, and a 3D capable data acquisition system (e.g. a Gen 3 
    Acquisition Board or ONIX).

..  raw:: html

    <center>
        <figure>
            <video width="560" height="340" controls muted>
                <source src="../_static/images/SPI_commutator.mp4" type="video/mp4">
            </video>
            <figcaption> 
                A demonstration of automated commutation using an <a href="url"
                src="https://open-ephys.org/acquisition-system/oeps-9029">
                Acquisition Board Gen 3</a> (not shown), a <a href="url"
                src="https://open-ephys.org/acquisition-system/oeps-6570-6571">
                Low-profile 3D capable 64ch headstage</a>, and an <a href="url"
                src="https://open-ephys.org/commutators/oeps-7761">SPI
                commutator</a>.
            </figcaption>
        </figure>
    </center>

Most acquisition systems rely on a tether to transmit power and data between the
headstage and the data acquisition controller. As the animal moves around during
freely behaving experiments, the tether can get twisted and tangled which risks
exerting torque on the animal. This can be is mitigated by using a commutator, a
device that untwists the tether as the animal moves around while maintaining
electrical continuity between the headstage on the animal and data acquisition
controller. Our commutators don't require tether torque measurements to know in
which direction to compensate for the twists. Instead, they use information from
absolute orientation sensors on our 3D capable headstages. Torque-free automatic
commutation relieves strain from the animal, encouraging more naturalistic
behaviors and enabling long-term recordings.

This tutorial demonstrates how to automate commutation using the Open Ephys
GUI, a 3D capable headstage, and an Open Ephys commutator. 

Hardware Configuration
#######################

..  tab-set::
    :sync-group: acquisition-hardware

    ..  tab-item:: Acquisition Board
        :sync: acquisition-board

        Make sure you have a `3D capable SPI headstage <https://open-ephys.github.io/acq-board-docs/Hardware-Guide/Headstages.html#open-ephys-headstages>`__ which have an Inertial Measurement Unit (IMU).

        #.  Follow the `Acquisition Board Quick Start Guide
            <https://open-ephys.github.io/acq-board-docs/User-Manual/Quickstart-guide.html#connecting-the-usb-cable-and-power-supply>`__
            to establish the following necessary acquisition board connections:

            -   USB 3.0 connection between the acquisition board and the PC.

            -   +5V connection between the acquisition board and an AC power
                source.

        #.  Follow the `SPI Commutator Connections section
            <https://open-ephys.github.io/commutator-docs/user-guide/mount-connect.html?commutator=spi#connecting>`__
            of the commutator hardware docs to establish the following necessary
            commutator connections:

            -   SPI connection between the commutator's stator and the
                `acquisition board <https://open-ephys.github.io/acq-board-docs/User-Manual/Quickstart-guide.html#connecting-the-headstages>`_.

            -   SPI connection between the commutator's rotor and the 3D capable
                headstage.
            
            -   USB connection between the commutator and the PC.   

    ..  tab-item:: ONIX
        :sync: onix

        Make sure you have a `3D capable ONIX headstage <https://open-ephys.github.io/onix-docs/Hardware%20Guide/Headstages/index.html>`__ which have an Inertial Measurement Unit (IMU), specifically, a BNO055 device.

        #.  Follow the `ONIX Hardware Guide
            <https://open-ephys.github.io/onix-docs/Hardware%20Guide/Breakout%20Board/setup.html>`__
            to establish the following necessary ONIX connections:

            -   2x (A & B) coaxial connections between the breakout board and
                the PCIe host.

            -   SDR connection between the breakout board and the PCIe host.

        #.  Follow the `Coax Commutator Connections section
            <https://open-ephys.github.io/commutator-docs/user-guide/mount-connect.html?commutator=coax#connecting>`__
            of the commutator hardware docs to establish the following necessary
            commutator connections:

            -   Coaxial connection(s) between the commutator's stator(s) and the
                acquisition board.

            -   Coaxial connection(s) between the commutator's rotor(s) and the 3D
                capable headstage.

            -   USB connection between the commutator and the PC.   

Software Configuration
####################################

#.  In the Open Ephys GUI, download the source processor for your hardware
    (:doc:`/User-Manual/Plugins/Acquisition-Board` or
    :doc:`/User-Manual/Plugins/Onix-Source`) via “File > Plugin Installer”.

#.  Download the signal chain that corresponds to which hardware you are using.

    ..  tab-set::
        :sync-group: acquisition-hardware

        ..  tab-item:: Acquisition Board
            :sync: acquisition-board

            :download:`Acquisition Board Commutator Signal Chain </_static/downloads/tutorials/commutator-signal-chain_acq-board>`

            ..  image:: /_static/images/tutorials/commutator/commutator-signal-chain_acq-board.webp
                :alt: Acquisition Board Signal Chain for commutation

        ..  tab-item:: ONIX
            :sync: onix

            :download:`ONIX Commutator Signal Chain </_static/downloads/tutorials/commutator-signal-chain_onix-source>`

            ..  image:: /_static/images/tutorials/commutator/commutator-signal-chain_onix-source.webp
                :alt: ONIX Signal Chain for commutation

#.  :ref:`Open <file>` the downloaded signal chain in the GUI.

    ..  tab-set::
        :sync-group: acquisition-hardware

        ..  tab-item:: Acquisition Board
            :sync: acquisition-board

            Confirm that "IMU" occupies one of the slots in headstage port
            indicator in the Acquisition Board processor after the
            Acquisition Board is initialized and headstage ports are
            scanned.

        ..  tab-item:: ONIX
            :sync: onix

            Confirm that one of the data devices on your headstage is a
            "BNO055" and that it is enabled using the processor's
            configuration canvas. 

#.  Refer to the :doc:`/User-Manual/Plugins/Commutator-Control` page to
    configure the Commutator Control processor.

    -   The selected Serial port should correspond to the COM port in which the commutator is connected. 

    -   The selected Stream should correspond to a 3D data stream. If multiple
        3D capable headstages are used, dual commutators, multiple 3D data
        streams could be available. Select the one you want to use. 

    -   For typical usage of an off-the-shelf Open Ephys 3D capable headstage,
        adjusting the rotation axis is not necessary. If you mount the headstage
        in a non-conventional location, refer to the `IMU Data
        <https://github.com/open-ephys/wiki/wiki/IMU-Data>`_ article and
        `channel maps docs <https://open-ephys.github.io/channel-maps-docs/headstages.html>`_
        for your particular hardware to figure out how to set the rotation axis.

#.  Make sure the GUI has connected to the acquisition system and click the ▶
    play button in the top-right corner. The commutator now follows the rotation
    of the headstage. 

