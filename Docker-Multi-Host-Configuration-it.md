Docker Multi Host Configuration
===============================
Docker supporta nativamente soltanto la comunicazione tra container in esecuzione sullo stesso host. Con la
configurazione qui descritta è possibile create un network virtuale che collega tra di loro tutti i container
in esecuzione su tutti gli host.

Per questo scenario sono usati:
- Oracle VM VirtualBox 5.0.
- Ubuntu Server 15.04.
- Docker (ultima versione).
- Open vSwitch (ultima versione).

Configurazione comune a tutti gli host
--------------------------------------
Una macchina virtuale con Ubuntu 15.04.

### Installazione
Docker (vedi http://docs.docker.com/linux/step_one/).

Open vSwitch
```bash
sudo apt-get install openvswitch-switch
```

### Container
Creazione di un container con
```bash
docker run -ai ubuntu bash
```

Docker collegherà interattivamente la sessione corrente con il container in esecuzione. Eseguire questa operazione
in una sessione ssh separata. Da questa sessione sarà possibile ispezionare il container e fare le prove di
collegamento.

Se il container è già stato creato, e non è in esecuzione, usare il comando
```bash
docker start -ai <nome container>
```

### Network Bridge
Occorre creare un bridge tra l'interfaccia di rete eth1 (internal network) e l'interfaccia di rete del container.

Creare il bridge con
```bash
sudo ovs-vsctl add-br br0
```

Aggiungere l'interfaccia eth1 al bridge con
```bash
sudo ovs-vsctl add-port br0 eth0
```

In questo modo è stato predisposto la prima parte del bridge per la creazione finale del virtual network.

Ubuntu Server 1
---------------
Configurare due schede di rete. Una scheda di rete per il network di gestione a cui collegarsi via ssh.
Una scheda di rete per il virtual nertwork che mette in comunicazione tra di loro i container.

### Adattatore di rete 1
PCnet-FAST III - NAT - Port Forwarding host port 50022 -> guest port 22
### Adattatore di rete 2
PCnet-FAST III - Internal Network (intnet) - Modalità promiscua in 'permetti tutto'
### Configurazione interfacce di rete sul server
/etc/network/interfaces
```
##Ubuntu Server 1
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
```

### Avvio del network virtuale
Con il container in esecuzione.

Aggiungere l'interfaccia di rete eth1 con
```bash
sudo ./ovs/utilities/ovs-docker add-port br0 eth1 <id container> --ipaddress=192.168.215.10/24
```

Impostare il network virtuale con
```bash
sudo ./ovs/utilities/ovs-docker set-vlan br0 eth1 <id container> 100
```

### Arresto del network virtuale
Per un arresto pulito del network virtuale occorre cancellare l'interfaccia eth1 dal container e dalla configurazione
del bridge dell'host.
```bash
sudo ./ovs/utilities/ovs-docker del-port br0 eth1 <id container>
```

Questa operazione è consigliabile farla perche l'interfaccia di rete eth1 sul container non soppravvive alla sua uscita.


Ubuntu Server 2
---------------
Configurare due schede di rete. Una scheda di rete per il network di gestione con a collegarsi via ssh.
Una scheda di rete per il virtual nertwork che mette in comunicazione tra di loro i container.

### Adattatore di rete 1
PCnet-FAST III - NAT - Port Forwarding host port 51022 -> guest port 22
### Adattatore di rete 1
PCnet-FAST III - Internal Network (intnet) - Modalità promiscua in 'permetti tutto'
### Configurazione interfacce di rete sul server
/etc/network/interfaces
```
##Ubuntu Server 2
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
```
### Avvio del network virtuale
Con il container in esecuzione.

Aggiungere l'interfaccia di rete eth1 con
```bash
sudo ./ovs/utilities/ovs-docker add-port br0 eth1 <id container> --ipaddress=192.168.215.11/24
```

Impostare il network virtuale con
```bash
sudo ./ovs/utilities/ovs-docker set-vlan br0 eth1 <id container> 100
```

### Arresto del network virtuale
Per un arresto pulito del network virtuale occorre cancellare l'interfaccia eth1 dal container e dalla configurazione
del bridge dell'host.
```bash
sudo ./ovs/utilities/ovs-docker del-port br0 eth1 <id container>
```

Questa operazione è consigliabile farla perche l'interfaccia di rete eth1 sul container non soppravvive alla sua uscita.

Test della configurazione
-------------------------
Se tutto funziona correttamente sarà possibile il ping tra i due container in esecuzione sui due host.

Sul container in esecuzione sull'host 1 provare con
```bash
ping 192.168.215.11
```

Sul container in esecuzione sull'host 2 provare con
```bash
ping 192.168.215.10
```
