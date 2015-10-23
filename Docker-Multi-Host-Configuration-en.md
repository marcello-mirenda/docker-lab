Docker Multi Host Configuration
===============================
Docker natively supports only communication from the container running on the same host. With the
configuration described here you can create a virtual network that connects them all containers
running on all hosts.

For this scenario they are used:
- Oracle VM VirtualBox 5.0.
- Ubuntu Server 15.04.
- Docker (last version).
- Open vSwitch (last version).

Configuration common to all hosts
--------------------------------------
A virtual machine with Ubuntu 15.04.

### Installation
Docker (see http://docs.docker.com/linux/step_one/).

Open vSwitch
```bash
sudo apt-get install openvswitch-switch
```

### Container
Creation of a container with
```bash
docker run -ai ubuntu bash
```

Docker interactively connect the current session with the container running. Do this
in a separate ssh session. From this session it will be able to inspect the container and rehearse
connection.

If the container has already been created, and is not running, use the command
```bash
docker start -ai <nome container>
```

### Network Bridge
We need to create a bridge between the network interface eth1 (internal network) and the network interface of the container.

Build the bridge with
```bash
sudo ovs-vsctl add-br br0
```

Add the eth1 interface with the bridge
```bash
sudo ovs-vsctl add-port br0 eth0
```

In this way it was prepared the first part of the bridge for the final creation of the virtual networks.

Ubuntu Server 1
---------------
Set up two network cards. A network card for network management to connect via ssh.
A network adapter for the virtual nertwork that connects each other containers.

### Network adapter 1
PCnet-FAST III - NAT - Port Forwarding host port 50022 -> guest port 22
### Network adapter 2
PCnet-FAST III - Internal Network (intnet) - Promiscuous mode in 'allow all'
### Configuring network interfaces on the server
/etc/network/interfaces
```
##Ubuntu Server 1
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
```

### Starting the virtual network
With the container running.

Aggiungere l'interfaccia di rete eth1 con
```bash
sudo ./ovs/utilities/ovs-docker add-port br0 eth1 <id container> --ipaddress=192.168.215.10/24
```

Add the network interface eth1
```bash
sudo ./ovs/utilities/ovs-docker set-vlan br0 eth1 <id container> 100
```

### Stopping the virtual network
For a clean shutdown of the virtual network must delete the interface eth1 from the container and configuration
of the host bridge.
```bash
sudo ./ovs/utilities/ovs-docker del-port br0 eth1 <id container>
```

This operation should be done because the network interface eth1 on the container does not survives to its termination.


Ubuntu Server 2
---------------
Set up two network cards. A network card for network management to connect via ssh.
A network adapter for the virtual nertwork that connects each other containers.

### Network adapter 1
PCnet-FAST III - NAT - Port Forwarding host port 51022 -> guest port 22
### Network adapter 2
PCnet-FAST III - Internal Network (intnet) - Promiscuous mode in 'allow all'
### Configuring network interfaces on the server
/etc/network/interfaces
```
##Ubuntu Server 1
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
```

### Starting the virtual network
With the container running.

Aggiungere l'interfaccia di rete eth1 con
```bash
sudo ./ovs/utilities/ovs-docker add-port br0 eth1 <id container> --ipaddress=192.168.215.11/24
```

Add the network interface eth1
```bash
sudo ./ovs/utilities/ovs-docker set-vlan br0 eth1 <id container> 100
```

### Stopping the virtual network
For a clean shutdown of the virtual network must delete the interface eth1 from the container and configuration
of the host bridge.
```bash
sudo ./ovs/utilities/ovs-docker del-port br0 eth1 <id container>
```

This operation should be done because the network interface eth1 on the container does not survives to its termination.

Testing the Configuration
-------------------------
If all goes well you can ping between the two containers running on two hosts.

The container running on the host 1 try
```bash
ping 192.168.215.11
```

The container running on the host 2 try
```bash
ping 192.168.215.10
```
