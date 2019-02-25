# TP 5 - Premier pas dans le monde Cisco

*Vous Pourrez retrouver le sujet [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/5) !*

# Sommaire

* [Matériel utilisé](#matériel-utilisé)
* [Préparation du lab !](#préparation-du-lab)
    * [Préparation des VMs](#préparation-des-VMs)
    * [Préparation de GNS](#préparation-de-GNS)
        * [Préparation des routeurs Cisco](#préparation-des-routeurs-cisco)
        * [Préparation du switches](#préparation-du-switches)
* [Topologie et tableau récapitulatif](#topologie-et-tableau-récapitulatif)
    * [Topologie](#topologie)
    * [Réseaux](#réseaux)
    * [Machines](#machines)


# Matériel utilisé

* **Virtualbox**
* **GNS3**

Les machines virtuelles Linux :

* l'OS est **CentOS 7** *(en version minimale)*.

Les routeurs Cisco :

* l'OS est **Cisco 3640**.

# Préparation du lab !

## Préparation des VMs

**Création des VMs**

Je clone des VMs depuis le patron du TP précédent :

* `server1.tp5.b1` est dans **net1** et porte l'IP `10.5.1.10/24`.

* `client1.tp5.b1` est dans **net2** et porte l'IP `10.5.2.10/24`.

* `client2.tp5.b1` est dans **net2** et porte l'IP `10.5.2.11/24`.

J'ajoutez aux trois VMs une **interface host-only** en deuxième carte dans le host-only précédemment créé.

## Préparation de GNS

* Edit > Preferences > VirtualBox VMs et ajouter les trois VMs.

Sur les 3 VMs

* Clic-droit > Configure > Network.
* Configurer 2 cartes réseau.

### Préparation des routeurs Cisco

Importez l'ISO du routeur et mettez-en deux dans GNS3 : 
* `router1.tp5.b1` est dans :
    * `net1` et porte l'IP `10.5.1.254/24`
    * `net12` et port l'IP `10.5.12.1` et le masque `255.255.255.0`. 
* `router2.tp5.b1` est dans :
    * `net2` et porte l'IP `10.5.2.254/24`
    * `net12` et port l'IP `10.5.12.2` et le masque `255.255.255.0`. 

### Préparation du Switches

Ici, on va utiliser les Ethernet Switches de GNS3 comme une multiprise. Un switch n'a pas d'IP.

:P

# Topologie et tableau récapitulatif

## Topologie
```
                                             client1
                                            /
Server1 --net1-- R1 --net12-- R2 --net2-- Sw
                                            \
                                             client2
```


## Réseaux

* `net1` : `10.5.1.0/24`
* `net2` : `10.5.2.0/24`
* `net12` : `10.5.12.0/24`

## Machines

Machine | `net1` | `net2` | `net12`
--- | --- | --- | ---
`client1.tp5.b1` | X | `10.5.2.10` | X
`client2.tp5.b1` | X | `10.5.2.11` | X
`router1.tp5.b1` | `10.5.1.254` | X | `10.5.12.1/24`
`router2.tp5.b1` | X | `10.5.2.254` | `10.5.12.2/24`
`server1.tp5.b1` | `10.5.1.10` | X | X

Tout est oké ! :)

# DHCP

On va recycler `client2.tp5.b1` pour ça :

* Renommer la machine

    pour porter le nom `dhcp-net2.tp5.b1`

*  Installer le serveur DHCP:
    * Ajout d'une carte nat dans la config réseau du (ancien) client2
    * la config enp0s9 est `OKEEEE`
    * `yum install -y dhcp`
    * redémarrer !
* Rallumer la VM dans GNS
* Configuration du serveur DHCP
    * => `cd /etc/dhcp/dhcpd.conf`
    
    Mettre la config du serveur *DHCP*
```
# dhcpd.conf

# option definitions common to all supported networks
option domain-name "tp5.b1";

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

subnet 10.5.2.0 netmask 255.255.255.0 {
  range 10.5.2.50 10.5.2.70;
  option domain-name "tp5.b1";
  option routers 10.5.2.254;
  option broadcast-address 10.5.2.255;
}
```
* Démarrer le serveur **DHCP**
    * `systemctl start dhcpd`
    * `systemctl status dhcpd` qui nous donne : `Active Running`
* Faire un **test**
    * connecter `client1` à `dhcp-net2`
    * Dans la config `ifcfg-enp0s3` je met le paramètree `BOOTPROTO = dhcp`
    * `ifdown enp0s3`
    * `ifup enp0s3`
    * `dhclient -v`

Encore une fois tout est oké !

*Par USEREAU Lucas*