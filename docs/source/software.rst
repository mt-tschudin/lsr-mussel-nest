Software
========

The software is distributed between a surface ``Brain Pi`` and an underwater
``Pod Pi``. The Brain Pi handles high-level commands, experiment scheduling,
and data storage, while the Pod Pi provides the interface to cameras and
actuators.


Architecture
------------

The system follows a client-server architecture:

::

    Operator
       │ SSH
       ▼
    Brain Pi
       │
       │ HTTP over Ethernet
       ▼
    Pod Pi
       │
       ├── Camera 1 ─ USB ─ XIAO ESP32-S3
       ├── Camera 2 ─ USB ─ XIAO ESP32-S3
       ├── Camera 3 ─ USB ─ XIAO ESP32-S3
       ├── Camera 4 ─ USB ─ XIAO ESP32-S3
       │
       ├── Propeller 1 ─ ESC / GPIO
       ├── Propeller 2 ─ ESC / GPIO
       │
       ├── UVC 1 ─ Relay / GPIO
       └── UVC 2 ─ Relay / GPIO

The Brain Pi determines **what should happen and when**. The Pod Pi translates
these high-level requests into local hardware actions.


Brain Pi
--------

The Brain Pi acts as the mission controller. Typical commands include:

* capture one or all cameras
* switch UVC channels on or off
* set a propeller PWM command
* stop a propeller
* request system status
* run scheduled experiment sequences

Main files:

::

    ~/lsr_control/
    ├── brain_control.py
    ├── pod_api.py
    ├── experiment.py
    ├── scheduler.py
    └── config.py

``brain_control.py`` provides manual control, while ``scheduler.py`` is used
for automatic experiment execution. Both access the Pod through ``pod_api.py``.


Pod Pi
------

The Pod Pi acts as the hardware server. It receives commands from the Brain Pi
and controls the devices locally.

Main files:

::

    ~/lsr_control/
    ├── pod_server.py
    ├── cameras.py
    ├── propellers.py
    ├── uv_lights.py
    ├── config.py
    └── data/

The Pod Pi handles camera communication over USB and controls the UVC relays
and propeller ESCs through GPIO.


API and Command Flow
--------------------

The Brain Pi communicates with the Pod Pi through an HTTP API.

Typical requests include:

::

    camera/1/capture
    uv/1/on
    uv/1/off
    propeller/1/set?pulse=1650
    status

The software path is:

::

    brain_control.py
          │
          ▼
       pod_api.py
          │ HTTP
          ▼
     pod_server.py
          │
          ├── cameras.py
          ├── uv_lights.py
          └── propellers.py

This separates high-level experiment logic from low-level hardware control.


Experiment Management
---------------------

Each experiment receives its own directory on the Brain Pi:

::

    ~/lsr_data/experiments/
    └── exp_20260821_005/
        ├── experiment.json
        ├── events.jsonl
        └── captures/
            ├── cam1/
            ├── cam2/
            ├── cam3/
            └── cam4/

``experiment.json`` stores the experiment configuration, while
``events.jsonl`` records what actually happened during execution.

Using an append-only JSONL log means completed events remain stored even if
an experiment is interrupted.


Creating an Experiment
----------------------

An experiment is created from the Brain Pi:

.. code-block:: bash

   python3 experiment.py create \
     --description "Integrated UV camera propeller test" \
     --notes "UV1 10-40 s; cameras start 12 s; propeller1 1550 us from 25-35 s" \
     --allow-actuation

For a camera-only experiment, omit ``--allow-actuation``.

The experiment module creates the directory structure, writes
``experiment.json``, prepares the camera folders, and initializes
``events.jsonl``.


Scheduler
---------

Manual and automatic control use the same Pod API:

::

                         MANUAL
                    brain_control.py
                          │
                          ▼
                       pod_api.py
                          │
                          ▼
                     pod_server.py


                       AUTOMATIC
                       scheduler.py
                          │
                     experiment.py
                          │
                       pod_api.py
                          │
                          ▼
                     pod_server.py

This allows the same tested hardware commands to be used interactively during
development and automatically during long-duration experiments.

========
