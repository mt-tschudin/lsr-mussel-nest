Network
=======

The system uses the Brain Pi as the network gateway between the operator and
the submerged Pod Pi. Only the Brain Pi connects to the deployment Wi-Fi;
the Pod Pi communicates with the Brain through a dedicated Ethernet link.


Network Architecture
--------------------

::

    Operator computer
          │
          │ Wi-Fi / SSH
          ▼
       Brain Pi
       wlan0 = deployment Wi-Fi
       eth0  = <BRAIN_INTERNAL_IP>
          │
          │ Ethernet
          ▼
        Pod Pi
       eth0  = <POD_INTERNAL_IP>

The Brain Pi provides the upstream network connection and forwards traffic to
the Pod Pi. The Pod Pi does not normally need to connect directly to Wi-Fi.


Deployment Wi-Fi
----------------

The Brain Pi stores the deployment Wi-Fi as an automatically connecting
NetworkManager profile.

Public documentation should not contain credentials. Use:

::

    SSID:       <DEPLOYMENT_WIFI_SSID>
    Password:   <NOT STORED IN PUBLIC DOCUMENTATION>
    Brain MAC:  <BRAIN_WLAN_MAC>

The deployment Wi-Fi should have a higher connection priority than the
fallback hotspot.

Example:

.. code-block:: bash

   sudo nmcli connection modify "<DEPLOYMENT_WIFI_PROFILE>" \
       connection.autoconnect yes \
       connection.autoconnect-priority 100

A fallback hotspot may be configured with a lower priority.


Brain-to-Pod Ethernet
---------------------

The Brain and Pod use a dedicated Ethernet network with fixed addresses.

::

    Brain eth0:  <BRAIN_INTERNAL_IP>
    Pod eth0:    <POD_INTERNAL_IP>

The Brain Ethernet interface uses a shared connection and acts as the gateway.
The Pod uses a static address with the Brain as gateway and DNS server.

The Brain Ethernet profile is configured to reconnect automatically after boot.


SSH Access
----------

The normal access path is:

::

    Computer → Brain Pi → Pod Pi

Connect to the Brain:

.. code-block:: bash

   ssh <USER>@<BRAIN_HOSTNAME>.local

If ``.local`` discovery is unavailable, use the Wi-Fi address assigned to the
Brain:

.. code-block:: bash

   ssh <USER>@<BRAIN_WIFI_IP>

From the Brain, connect to the Pod:

.. code-block:: bash

   ssh <USER>@<POD_INTERNAL_IP>


Connection Check
----------------

On the Brain:

.. code-block:: bash

   nmcli device status
   ip -br addr show wlan0
   ip -br addr show eth0
   ping -c 3 <POD_INTERNAL_IP>

On the Pod:

.. code-block:: bash

   ip -br addr show eth0
   ping -c 3 <BRAIN_INTERNAL_IP>
   ping -c 3 8.8.8.8
   ping -c 3 raspberrypi.com

A successful check confirms:

* operator → Brain communication
* Brain → Pod communication
* Pod → Brain communication
* Pod Internet access through the Brain
* DNS resolution on the Pod


Troubleshooting
---------------

If the physical Ethernet link is active but the Brain has no internal IP
address, verify that the correct NetworkManager Ethernet profile is active:

.. code-block:: bash

   nmcli device status
   ip -br addr show eth0

If necessary, reactivate the configured Brain Ethernet profile:

.. code-block:: bash

   sudo nmcli connection up <BRAIN_ETHERNET_PROFILE>

Avoid manually adding an IP address with ``ip addr add`` as a permanent fix.
Persistent configuration should remain managed by NetworkManager.

=======
