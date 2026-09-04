Deployment
==========

This section summarizes the procedure for preparing, starting, validating,
and shutting down the LSR Mussel Nest system at the deployment site.

.. raw:: html

   <video controls style="width:50%; max-width:800px; display:block; margin:auto;">
     <source src="_static/images/deployment/Deployment_Test_1_web.mp4" type="video/mp4">
   </video>

.. raw:: html

   <video controls style="width:50%; max-width:800px; display:block; margin:auto;">
     <source src="_static/images/deployment/Deployment_Test_2.mp4" type="video/mp4">
   </video>

Before Deployment
-----------------

Before powering the system, verify:

* waterproof enclosure closed and inspected
* WetLink penetrators fully tightened
* camera housings closed
* Ethernet tether connected
* power connections secured
* propellers mechanically free
* UVC lamps correctly mounted
* deployment Wi-Fi available
* Brain Pi network profile already configured

Do not store Wi-Fi passwords or other credentials in the public documentation.


Startup Procedure
-----------------

1. Connect the Ethernet link between Brain Pi and Pod Pi.
2. Power the Brain Pi.
3. Wait approximately 30–60 seconds for Wi-Fi and Ethernet initialization.
4. Connect the operator computer to the same deployment Wi-Fi.
5. Connect to the Brain Pi:

   .. code-block:: bash

      ssh <USER>@<BRAIN_HOSTNAME>.local

6. Verify the Brain network:

   .. code-block:: bash

      nmcli device status
      ip -br addr show wlan0
      ip -br addr show eth0

7. Power or verify the Pod Pi.
8. From the Brain, check communication with the Pod:

   .. code-block:: bash

      ping -c 3 <POD_INTERNAL_IP>

9. Connect to the Pod:

   .. code-block:: bash

      ssh <USER>@<POD_INTERNAL_IP>


Start Pod Services
------------------

The Pod hardware interface must be running before experiments are started.

Start the GPIO daemon:

.. code-block:: bash

   sudo pigpiod

Then start the Pod API server:

.. code-block:: bash

   cd ~/lsr_control
   python3 pod_server.py

Keep the server running during operation.


System Check
------------

From the Brain Pi, verify that the Pod API responds:

.. code-block:: bash

   curl http://<POD_INTERNAL_IP>:8000/status

or:

.. code-block:: bash

   cd ~/lsr_control
   python3 -c "import pod_api; print(pod_api.get_status())"

Before a long deployment, test the subsystems individually:

* capture one image
* capture all cameras
* switch each UVC channel briefly
* run each propeller at a low test command
* return both propellers to neutral
* request system status


Create an Experiment
--------------------

Experiments are created on the Brain Pi.

Example:

.. code-block:: bash

   python3 experiment.py create \
       --description "Integrated camera and stimulation test" \
       --notes "Deployment test sequence" \
       --allow-actuation

For a monitoring-only experiment, omit ``--allow-actuation``:

.. code-block:: bash

   python3 experiment.py create \
       --description "Camera-only monitoring"

Each experiment receives its own directory containing its configuration,
event log, and captured images.


Experiment Data
---------------

Experiment data is stored on the Brain Pi:

::

    ~/lsr_data/experiments/
    └── <EXPERIMENT_ID>/
        ├── experiment.json
        ├── events.jsonl
        └── captures/
            ├── cam1/
            ├── cam2/
            ├── cam3/
            └── cam4/

``experiment.json`` records the intended configuration.

``events.jsonl`` records the actions that actually occurred during the
experiment. Because events are appended individually, previously written
entries remain available if execution is interrupted.


Pre-Deployment Functional Test
------------------------------

Before submerging the complete system, perform a dry functional test:

* Brain ↔ Pod communication
* camera capture and image retrieval
* LED illumination
* UVC relay switching
* propeller PWM control
* status reporting

After the dry test, perform a short wet test before committing to an extended
deployment.


Safe Shutdown
-------------

Stop active experiments and return actuators to their safe state before
shutting down.

On each Raspberry Pi:

.. code-block:: bash

   sudo shutdown now

Wait until shutdown has completed before disconnecting power.


Deployment Checklist
--------------------

Before leaving the system unattended, confirm:

* Brain connected to deployment Wi-Fi
* Brain Ethernet interface active
* Brain → Pod ping successful
* Brain → Pod SSH successful
* Pod API responding
* cameras responding
* UVC channels responding
* propellers responding and returning to neutral
* experiment directory created
* image storage path verified
* event logging active
