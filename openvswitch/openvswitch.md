!SLIDE
# OpenVSwitch

Créé par les fondateurs de Nicira (racheté par VMWare).

!SLIDE
# Historique

* 1.0 : 17/05/2010
* 1.1 : 05/04/2011
* 1.2 : 03/08/2011
* 1.3 : 09/12/2011
* 1.4 : 30/01/2012
* 1.5 : 01/06/2012
* 1.7 : 30/07/2012
* 1.9 : 26/03/2013

!SLIDE
# Fonctionnalités

* Standard 802.1Q VLAN model with trunk and access ports
* NIC bonding with or without LACP on upstream switch
* NetFlow, sFlow(R), and mirroring for increased visibility
* QoS (Quality of Service) configuration, plus policing
* GRE, GRE over IPSEC, VXLAN, and LISP tunneling
* 802.1ag connectivity fault management
* OpenFlow 1.0 plus numerous extensions
* Transactional configuration database with C and Python bindings
* High-performance forwarding using a Linux kernel module

!SLIDE
# Limitation à OpenFLow 1.0

Gère que le protocole 1.0 :

* Pas IPv6
* Pas de tags MPLS

Objectifs 1.1+ :

https://github.com/osrg/openvswitch/blob/master/OPENFLOW-1.1%2B

!SLIDE
# Composants

Découpé en 2 parties :

* Module noyau (intégré dans la 3.3+)
* Outils userland

!SLIDE
# Outils

OpenVSwitch est découpée en plusieurs outils :

* ovsdb-server (gestion de la BD)
* ovs-vswitchd (démon qui implémente le switch)
* ovs-dpctl (configuration du module noyau)
* ovs-vsctl (configuration du switch)
* ovs-ofctl (requête et contrôle des switchs)
* ovs-appctl (envoyer des commandes aux démons)
* ovsdbmonitor (outil GUI pour visualiser BD / table de flow)
* ovs-controller (contrôleur simple de démo)

!SLIDE
# Plateformes

Plateformes supportées :

* KVM
* VirtualBox
* Xen / Xen Cloud Platform / XenServer

!SLIDE
# Défaut majeur

Manque d'intégration avec Linux.

!SLIDE
# Base de données

BD au format JSON :

* table Open_vSwitch (configuration OVS)
* table Bridge
* table Port
* table Interface
* table Mirror
* table ...

!SLIDE
# ovs-switchd

Gère le switch :

* 256 bridges maximun
* 65280 ports / bridge
* 2048 MAC / bridge
* 32 mirroirs / bridge
* Performance dégragée après 1M de flux par bridge

!SLIDE
# ovs-vsctl

Utilitaire de requête et de configuration ovs-switchd (interface de haut niveau à la BD).

* add-br / del-br /list-br
* add-port / del-port / list-port
* add-bond
* list-ifaces
* ...

!SLIDE
# ovs-vsctl

API bas niveau pour manipuler la BD :

* add table record column[:key]=value
* set table record column[:key]=value
* remove table record column key
* ...

Notes :

* Possible de créer un fake-bridge
* Possibilité de configurer localement ou par un contrôleur

!SLIDE
# Exemple simple

Ajouter eth0 et tap0 (VLAN 9) dans le bridge :

```bash
$ ovs-vsctl add-br br0
$ ovs-vsctl add-port br0 eth0
$ ovs-vsctl add-port br0 tap0 tag=9
```

!SLIDE
# Exemple de miroir de port

```bash
$ ovs-vsctl add-br br0
$ ovs-vsctl add-port br0 eth0
$ ovs-vsctl add-port br0 tap0
$ ovs-vsctl add-port br0 tap1 \
    -- --id=@p get port tap1 \
    -- --id=@m create mirror name=m0 select-all=true output-port=@p
    -- set bridge br0 mirrors=@m
```

!SLIDE
# Exemple de miroir avec VLAN

br0 avec eth0 comme trunk et tap0 pour le VLAN 10. Tout le trafic tap0 + VLAN 10 sur tap0 est mirroré sur le VLAN 15 sur eth0.

```bash
$ ovs-vsctl add-br br0
$ ovs-vsctl add-port br0 eth0
$ ovs-vsctl add-port br0 tap0 tag=10
$ ovs-vsctl -- --id=@m create mirror name=m0 select-all=true \
    select-vlan=10 output-vlan=15 \
    -- set bridge br0 mirrors=@m
```

!SLIDE
# Mirroring dans un tunnel GRE

Mirroir de eth0 et tap0 dans le tunnel GRE (ERSPAN non supporté)

```bash
$ ovs-vsctl add-br br0
$ ovs-vsctl add-port br0 eth0
$ ovs-vsctl add-port br0 tap0
$ ovs-vsctl add-port br0 gre0 \
    -- set interface gre0 type=gre options:remote_ip=192.168.1.10 \
    -- --id=@p get port gre0 \
    -- --id=@m create mirror name=m0 select-all=true output-port=@p \
    -- set bridge br0 mirrors=@m
```

!SLIDE
# SFLOW

Générer du trafic sflow :

```bash
$ ovs-vsctl — –id=@sflow create sflow agent=eth1 target=\”10.0.0.1:6343\”
                                header=128 sampling=64 polling=10
            — set bridge br0 sflow=@sflow
```

!SLIDE
# QOS

Limiter tap0 à 1Mo et 100Ko/s :

```bash
$ ovs-vsctl set Interface tap0 ingress_policing_rate=1000
$ ovs-vsctl set Interface tap0 ingress_policing_burst=100
```

!SLIDE
# ocs-ofctl

Administration des switchs OpenFlow. Permet de voir l'état courant d'un switch.

* show
* dump-tables / dump-ports / mod-ports / dump-flows
* add-flow / add-flows / mod-flows / del-flows

!SLIDE
# Flow syntax

Caractéristiques L2 / L3 :

* port du switch
* VLAN
* IP src / dst
* MAC src / dst
* ethertype
* protocole
* TOS
* TTL
* code / type ICMP
* fragmentation
* tunnel

!SLIDE
# Action

Plusieurs actions quand un paquet matche :

* port de sortie
* drop
* queue
* modifier une caractéristique
* learn
* ...
