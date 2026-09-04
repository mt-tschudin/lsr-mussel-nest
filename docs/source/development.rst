Development
===========

The LSR Mussel Nest was designed as a modular research platform that can be
extended as the experimental requirements evolve.

This section summarizes the software organization, recommended development
workflow, and the main directions for future work.


Repository Structure
--------------------

On the Raspberry Pis, the active control software is stored under:

::

    ~/lsr_control/

while experiment data is stored separately under:

::

    ~/lsr_data/experiments/


Development Workflow
--------------------

Changes should normally be developed and tested in stages:

1. test the individual hardware function
2. test the corresponding Pod API command
3. test the command from the Brain Pi
4. integrate it into an experiment sequence
5. validate the complete system before deployment

This separation helps keep low-level hardware changes from affecting the
experiment logic unexpectedly.


Adding New Hardware
-------------------

New actuators or sensors should be integrated through the Pod Pi.

The recommended pattern is:

::

    Brain logic
        │
        ▼
    pod_api.py
        │
        ▼
    pod_server.py
        │
        ▼
    device module
        │
        ▼
    hardware

A new device should therefore have:

* a dedicated hardware module
* configuration entries in ``config.py``
* one or more API endpoints
* a corresponding Brain-side function
* a simple bench test before scheduler integration


Adding New Experiments
----------------------

Experiment logic should remain on the Brain Pi.

The experiment layer is responsible for:

* creating an experiment directory
* storing experiment metadata
* defining capture destinations
* logging completed events
* calling the Pod API

This keeps experiment design separate from the hardware implementation.


Configuration
-------------

Hardware-specific values should be kept in configuration files rather than
hard-coded into the control logic.

Typical configuration values include:

* GPIO assignments
* camera identifiers
* actuator limits
* storage paths
* API settings
* safe default values

Sensitive values such as passwords, Wi-Fi credentials, tokens, or private keys
should not be stored in the public repository.


Known Limitations
-----------------

The current system still has several limitations:

* the biological effectiveness of UVC and hydrodynamic stimulation remains to
  be validated in long-term field experiments
* the current system operates in open loop
* long-duration reliability in Lake Geneva still requires further validation
* camera housings and test surfaces may themselves become fouled
* stimulation parameters still require biological calibration


Future Work
-----------

The main development directions are:

* long-term field deployment at LéXPLORE
* improved experiment scheduling
* automated recovery after interrupted experiments
* remote health monitoring
* camera-based colonization detection
* biofilm sensing
* closed-loop stimulation
* additional stimulation or sensing modules

The longer-term goal is to move from a remotely operated monitoring and
stimulation platform toward an autonomous system that can detect early
colonization and react accordingly.
