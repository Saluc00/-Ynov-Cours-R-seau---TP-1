# TP 3 - Plusieurs réseaux : routage statique

*Vous pourrez retrouver le sujet* [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/3) !

# Sommaire

* [Matériel utilisé](#matériel-utilisé)
* [Initiation](#initiation)
    * [Avant de commencer](#avant-de-commencer)
        * [Configuration réseau d'une machine CentOS](#configuration-réseau-dune-machine-centos)
        * [Utilisez une commande pour prouver que vous avez internet depuis la VM](#utilisez-une-commande-pour-prouver-que-vous-avez-internet-depuis-la-vm)
        * [Prouvez que votre PC hôte et la VM peuvent communiquer](#prouvez-que-votre-pc-hôte-et-la-vm-peuvent-communiquer)
        * [Affichez votre table de routage sur la VM et expliquez chacune des lignes](#affichez-votre-table-de-routage-sur-la-vm-et-expliquez-chacune-des-lignes)
        * [Faire joujou avec quelques commandes](#faire-joujou-avec-quelques-commandes)
            * [Ping](#Ping)
            * [Afficher la table de routage](#afficher-la-table-de-routage)
* [Notion de ports et SSH](#notion-de-ports-et-ssh)
    * [Exploration des ports locaux](#exploration-des-ports-locaux)
    * [SSH](#ssh)
    * [Firewall](#firewall)
        * [SSH](#ssh)
        * [Netcat](#netcat)
* [Routage statique](#routage-statique)
    * [Préparation des hôtes (vos PCs)](#Préparation-des-hôtes-(vos-pcs))
        * [Préparation avec câble](#préparation-avec-câble)
        * [Préparation VirtualBox](#préparation-virtualbox)
        * [Activation du routage sur les PCs](#Activation-du-routage-sur-les-pcs)
    * [Configuration du routage](#Configuration-du-routage)
        * [PC1](#pc1)
        * [PC2](#pc2)
    * [Activation du routage sur les VM](#Activation-du-routage-sur-les-VM)
        * [VM1](#vm1)
        * [VM2](#vm2)
* [Configuration des noms de domaines](#Configuration-des-noms-de-domaines)

# Matériel utilisé

Pour se *TP* j'utilise une **machine virtuelle**.

**Logiciel** : `VirtualBox`
**OS** : `centOS 7 (minimal)`
**PC** : 
    * `Mon PC personel`
    * `Ce magnifique pc dit *PORTABLE*`
    ![BeauPC](https://github.com/Saluc00/TP-Reseau-B1/blob/master/TP/3/ressources/beauPC.jpg)

# Initiation

Dans notre TP, j'utilise **une machine virtuelle** (*VM*) avec un *OS* [centOS](https://fr.wikipedia.org/wiki/CentOS).

### Avant de commencer
---

La *VM* configuré, il nous ai demandé de *désactiver* `SElinux`.

Pour cela,  j'ai entré :
* `sudo setenforce 0`
* `cd /etc/selinux`
* `vi config`
* modifié `SELINUX=permissive`

J'ai aussi ajouté une seconde carte réseau.
La première étant configuré en **NAT**.
La deuxième est alors en **Host-only**.

La carte NAT est automatiquement parametré.

Pour la carte en Host-only, c'est différent.

Il me faut créer son fichier de configuration dans à l'adresse : `/etc/sysconfig/network-scripts/`.

La dedans, j'ouvre **VIM** en faisant la commande `vi ifcfg-ens37`, puis inscrire la config de la carte réseau :

    TYPE=ETHERNET
    BOOTPROTO=static
    NAME=ens37
    DEVICE=ens37
    
    IDADDR=192.168.126.10
    NETMASK=255.255.255.0
    [it4@localhost network-scripts]$

Petit `ifdown ens37` et `ifup ens37` pour redemarrer la carte réseau *(Pour etre sur que la config sois correctement mise)*.

Avec `ip a` je peut voir la config de toutes les carte réseau.

Soit :

    1: lo <LOOPBACK,UP,LOWER> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    
    2: ens33: <BROADCAST,MULTCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:69:fa:df brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic enp0s3
        valid_lft 1794sec preferred_lft 1794sec
    inet6 fe80:af9d:238a:2fba:3e43/64 scope link noprefixroute
        valid_lft forever preferred_lft forever

    3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:69:fa:e9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.126.10/24 brd 192.168.126.255 scope global noprefixroute enp0s8
        valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe69:fae9/64 scope link
        valid_lft forever preferred_lft forever

### Configuration réseau d'une machine CentOS
---

A faire..

#### Utilisez une commande pour prouver que vous avez internet depuis la VM
---

Afin de prouver que j'ai accès à *internet*, je fais la commande `curl www.google.com`.

*Pour le coup, j'affiche pas le resultat.. un peu long à recopier. :/*
*Mais j'ai eu un gros bloc d'html !*

#### Prouvez que votre PC hôte et la VM peuvent communiquer
---

Pour ***prouver*** que j'ai bien un liaison entre mon pc et la *VM*.
Je fais un **ping** depuis mon pc vers la *VM* avec la commande `ping 192.168.126.10`.

    ping 192.168.126.10

    Envoi d’une requête 'Ping'  192.168.126.10 avec 32 octets de données :
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64

    Statistiques Ping pour 192.168.126.10:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms

Puis je **ping** l'hote depuis la *VM* en faisant `ping 192.168.126.1` sur la *machine virtuel*.

    ping 192.168.126.1 

    PING 192.168.126.1 (192.168.126.1) 56(84) bytes of data.
    64 bytes from 192.168.126.1: icmp_seq=1 ttl=128 time=0.277 ms
    64 bytes from 192.168.126.1: icmp_seq=2 ttl=128 time=0.248 ms
    64 bytes from 192.168.126.1: icmp_seq=3 ttl=128 time=0.212 ms
    64 bytes from 192.168.126.1: icmp_seq=4 ttl=128 time=0.251 ms

    --- 192.168.126.1 ping statistics ---
    6 packets transmitted, 6 received, 0% packet loss, time 5005ms
    rtt min/avg/max/mdev = 0.212/0.250/0.277/0.023 ms

*Bien sur avant de faire cette commande, la configuration de la carte réseau host-only est* `192.168.126.1` *avec un masque de sous réseau* `255.255.255.0`.

#### Affichez votre table de routage sur la VM et expliquez chacune des lignes

Maintenant pour afficher la table de routage, je fais la commande `ip route`.

    default via 10..0.2.2 dev enp0s3 proto dhcp metric 101
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 101
    192.168.126.0/24 dev enp0s8 proto kernel scope link src 192.168.126.10 metric 100

### Faire joujou avec quelques commandes
---

A faire..

#### Ping
---

*On peut voir les ping depuis le PC vers la VM et depuis la VM vers le PC au dessus à* ***Prouvez que votre PC hôte et la VM peuvent communiquer***

#### Afficher la table de routage 
---

*La table de routage de la VM est à* ***affichez votre table de routage sur la VM et expliquez chacune des lignes***

La table de routage de l'hôte est :

    ===========================================================================
    Liste d'Interfaces
    3...10 65 30 81 78 90 ......Killer E2500 Gigabit Ethernet Controller
    26...0a 00 27 00 00 1a ......VirtualBox Host-Only Ethernet Adapter
    24...02 00 4c 4f 4f 50 ......Npcap Loopback Adapter
    12...9e b6 d0 6b 1b 97 ......Microsoft Wi-Fi Direct Virtual Adapter
    20...ae b6 d0 6b 1b 97 ......Microsoft Wi-Fi Direct Virtual Adapter #2
    15...00 50 56 c0 00 01 ......VMware Virtual Ethernet Adapter for VMnet1
    19...00 50 56 c0 00 08 ......VMware Virtual Ethernet Adapter for VMnet8
    7...9c b6 d0 6b 1b 97 ......Killer Wireless-n/a/ac 1435 Wireless Network Adapter
    1...........................Software Loopback Interface 1
    ===========================================================================

    IPv4 Table de routage
    ===========================================================================
    Itinéraires actifs :
    Destination réseau    Masque réseau  Adr. passerelle   Adr. interface Métrique
            0.0.0.0          0.0.0.0      10.33.3.253      10.33.2.249     35
            10.33.0.0    255.255.252.0         On-link       10.33.2.249    291
        10.33.2.249  255.255.255.255         On-link       10.33.2.249    291
        10.33.3.255  255.255.255.255         On-link       10.33.2.249    291
            127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
            127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
    127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
        169.254.0.0      255.255.0.0         On-link    169.254.53.133    281
    169.254.53.133  255.255.255.255         On-link    169.254.53.133    281
    169.254.255.255  255.255.255.255         On-link    169.254.53.133    281
        192.168.126.0    255.255.255.0         On-link     192.168.126.1    281
        192.168.126.1  255.255.255.255         On-link     192.168.126.1    281
    192.168.126.255  255.255.255.255         On-link     192.168.126.1    281
        192.168.127.0    255.255.255.0         On-link    192.168.127.10    291
    192.168.127.10  255.255.255.255         On-link    192.168.127.10    291
    192.168.127.255  255.255.255.255         On-link    192.168.127.10    291
        192.168.234.0    255.255.255.0         On-link     192.168.234.1    291
        192.168.234.1  255.255.255.255         On-link     192.168.234.1    291
    192.168.234.255  255.255.255.255         On-link     192.168.234.1    291
            224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
            224.0.0.0        240.0.0.0         On-link     192.168.126.1    281
            224.0.0.0        240.0.0.0         On-link       10.33.2.249    291
            224.0.0.0        240.0.0.0         On-link    192.168.127.10    291
            224.0.0.0        240.0.0.0         On-link    169.254.53.133    281
            224.0.0.0        240.0.0.0         On-link     192.168.234.1    291
    255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
    255.255.255.255  255.255.255.255         On-link     192.168.126.1    281
    255.255.255.255  255.255.255.255         On-link       10.33.2.249    291
    255.255.255.255  255.255.255.255         On-link    192.168.127.10    291
    255.255.255.255  255.255.255.255         On-link    169.254.53.133    281
    255.255.255.255  255.255.255.255         On-link     192.168.234.1    291
    ===========================================================================
    Itinéraires persistants :
    Adresse réseau    Masque réseau  Adresse passerelle Métrique
            0.0.0.0          0.0.0.0    192.168.127.1  Par défaut
    ===========================================================================

    IPv6 Table de routage
    ===========================================================================
    Itinéraires actifs :
    If Metric Network Destination      Gateway
    1    331 ::1/128                  On-link
    26    281 fe80::/64                On-link
    7    291 fe80::/64                On-link
    15    291 fe80::/64                On-link
    24    281 fe80::/64                On-link
    19    291 fe80::/64                On-link
    7    291 fe80::83e:edbd:4ed1:ee06/128
                                        On-link
    19    291 fe80::cfc:d6a7:2b45:9467/128
                                        On-link
    26    281 fe80::a5b3:39f7:2165:7b03/128
                                        On-link
    15    291 fe80::cd11:281c:8caf:8650/128
                                        On-link
    24    281 fe80::e102:3ef5:c3bf:3585/128
                                        On-link
    1    331 ff00::/8                 On-link
    26    281 ff00::/8                 On-link
    7    291 ff00::/8                 On-link
    15    291 ff00::/8                 On-link
    24    281 ff00::/8                 On-link
    19    291 ff00::/8                 On-link
    ===========================================================================
    Itinéraires persistants :
    Aucun

#### Mettre en évidence la ligne qui leur permet de discuter via le réseau host-only (dans chacune des tables)
---

* hôte

    192.168.126.0    255.255.255.0         On-link     192.168.126.1    281
    192.168.126.1  255.255.255.255         On-link     192.168.126.1    281
    192.168.126.255  255.255.255.255         On-link     192.168.126.1    281

* VM

    192.168.126.0/24 dev enp0s8 proto kernel scope link src 192.168.126.10 metric 100

#### Depuis la VM utilisez curl (ou wget) pour télécharger un fichier sur internet
---
*On peut voir cela ici :* ***utilisez une commande pour prouver que vous avez internet depuis la VM***.

#### depuis la VM utilisez dig pour connaître l'IP 
---

Avant tout, il me faut télécharger bind-utils pour utiliser la commande `dig` en faisant `yum install bind-utils`.

* Pour **Ynov.com**

Avec la commande `dig ynov.com`.

Il me dit que *l'ip* du serveur d'Ynov est `217.70.184.38`.

* Pour **Google.com**

Avec la commande `dig google.com`.

je peut voir que *l'ip* du serveur de Google est `216.58.204.238`.

## Notion de ports et SSH

### Exploration des ports locaux
---

En faisant `ss -al4tnp`.

il m'affiche ça :

    State      Recv-Q Send-Q               Local Address:Port                              Peer Address:Port
    LISTEN     0      128                              *:22                                           *:*
    LISTEN     0      100                      127.0.0.1:25                                           *:*

Soit :

* a : affiche tout les ports.
* l : affiche les port qui écoute.
* 4 : pour avoir uniquement les connexions en *IPv4*.
* t : affiche les port qui utilise le protocole `TCP`.
* n : pour ne pas transformer le numéro de ports en nom de service.
* p : pour connaître l'application qui écoute sur ce port.

### SSH
---

*SSH est un protocole pour se connecter sur un serveur à distance.*

Sur : https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

J'ai téléchargé **putty.exe (the SSH and Telnet client itself)**.
Une fois ouvert, j'inscris l'ip de la *VM* (soit : `192.168.126.10`) puis je me *log*.

**C'est bon !!!!** Je suis connecté !

### Firewall
---

#### SSH 
---

* J'ai modifier le fichier /etc/ssh/sshd_config.
    * Changer le numéro du port sur lequel votre serveur SSH écoute, *soit* `2222`.
* Redémarrez le serveur SSH pour que le changement prenne effet.
    * Avec `systemctl restart sshd`
* En faisant sn -4ln, je peut voir que le port **SSH** est changé par `2222`.
* Maintenant, j'ouvrir `Putty.exe`.
    * Je coche **SSH**.
    * Change le port `22` (port ssh par defaut) par `2222`.
    * Go ! 

Marche pô..

Ok.. **Qu'est ce que pourquoi ça ne pas marcher ?**

J'ai changé le port ? Mais je ne l'ai pas *autorisé* dans le **firewall**.

Pour changer le port : `firewall-cmd --add-port=2222/tcp --permanent` puis `firewall-cmd --reload`pour redemarrer le service.

#### Netcat
---

Il nous faut un `serveur` et un `client`

La *VM* sera le `serveur`.
Putty sera alors le `client`.

*Putty étant connecté au `ssh` de la VM*.

* Sur la VM:

Installer **netcat** avec : `yum install nc`
puis `nc -l -p 5454`.

Serveur initialisé !

* Sur putty:

Faire `telnet 192.168.126.10 5454`.

Cha marche !

## Routage statique

Objectif :

    Internet            Internet
        |                    |
      WiFi                  WiFi
        |                    |
      PC 1  ---Ethernet--- PC 2
        |                    |
    Host-only 1          Host-only 2
        |                    |
      VM 1                 VM 2

C'est à dire, interconnecter le PC (PC1) hôte avec un second PC (PC2).
Sur le second PC (PC2), mettre un VM (VM2).

## Préparation des hôtes (vos PCs)

#### Préparation avec câble
---

Faire en sorte que :

    * Les deux PCs puissent se ping à travers le câble.
    * Les cartes Ethernets doivent être dans le réseau 12 : 192.168.112.0/30.

#### Préparation VirtualBox
---

Modifier les réseaux host-only pour qu'ils soient :

* PC1 : réseau 1 : 192.168.101.0/24
* PC2 : réseau 2 : 192.168.102.0/24

Modifiez l'adresse des VM :

* VM1 (sur PC1) : 192.168.101.10
* VM2 (sur PC2) : 192.168.102.10

#### Activation du routage sur les PCs
---

Vue que les deux PCs hôtes sont des *windows*, je fais les manipulations suivantes sur le PC1 et 2 :

* modifier la clé registre `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\ Services\Tcpip\Parameters\IPEnableRouter` en passant sa valeur de `0` à `1`
* vous pouvez reboot là (ça évite des pbs)
* activer le service de routage ("lancer l'application de routage")
    * `Win + R`
    * `services.msc` > `Entrée`
    * Naviguer à "Routage et accès distant"
        * Clic-droit > Propriétés
        * Pour le démarrage, sélectionner "Automatique"
        * `Appliquer`
        * `Démarrer`

**bilan des adresses IP portés par chacune des interfaces utilisées dans le TP**

### Configuration du routage

J'ai 3 réseaux : 
- **réseau `1`**, un /24 entre *PC1* et *VM1* (host-only 1)
Qui contient : 
    * PC1 : `192.168.101.1`
    * VM1 : `192.168.101.10`
- **réseau `2`**, un /24 entre *PC2* et *VM2* (host-only 2)
Qui contient :
    * PC2 : `192.168.102.1`
    * VM2 : `192.168.102.10`
- **réseau `12`**, un /30 entre *PC1* et *PC2* (câble)
    * PC1 : `192.168.112.1`
    * PC2 : `192.168.112.2`

#### PC1
---
Pour faire le *routage* sur le PC1 entre le réseau du PC1 et la VM1 **et** au réseau du PC1 et PC2, je tape : `route add 192.168.102.0/24 mask 255.255.255.0 192.168.112.2`.

Le PC1 ping maintenant le *PC2* sur l'ip `192.168.102.1` !

#### PC2 
---
Pour faire le *routage* sur le *PC2* entre le réseau du PC2 et la VM2 **et** au réseau du PC1 et PC2, je tape : `route add 192.168.101.0/24 mask 255.255.255.0 192.168.112.1`.

Le PC2 ping maintenant le *PC1* sur l'ip `192.168.101.1` !

### Activation du routage sur les VM
---

#### VM1
---

Pour que la *VM1* puisse ping les PCs sur le réseau `192.168.112.0`, je fais :

    ip route add 192.168.112.0/24 via 192.168.101.1 dev enp0s8

Puis pour que la *VM1* puisse ping les PCs sur le réseau `192.168.102.0`, je fais :

    ip route add 192.168.102.0/24 via 192.168.101.1 dev enp0s8

Les ping fonctionnent !

#### VM2
---

Pour que la *VM1* puisse ping les PCs sur le réseau `192.168.112.0`, je fais :

    ip route add 192.168.112.0/24 via 192.168.102.1 dev enp0s8

Puis pour que la *VM1* puisse ping les PCs sur le réseau `192.168.102.0`, je fais :

    ip route add 192.168.101.0/24 via 192.168.102.1 dev enp0s8

Les ping fonctionnent !

## Configuration des noms de domaines

Apres avoir changé les noms de domaines..

Ping de la *VM1* au *PC1* :

    [root@localhost etc]# ping pc1.tp3.b1
    PING pc1 (192.168.112.1) 56(84) bytes of data.
    64 bytes from pc1 (192.168.112.1): icmp_seq=1 ttl=127 time=0.237 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=2 ttl=127 time=0.477 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=3 ttl=127 time=0.461 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=4 ttl=127 time=0.504 ms


Ping de la *VM* 1 au *PC2* :

    [root@localhost etc]# ping pc2.tp3.b1
    PING pc2 (192.168.112.2) 56(84) bytes of data.
    64 bytes from pc2 (192.168.112.2): icmp_seq=1 ttl=127 time=2.66 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=2 ttl=127 time=2.78 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=3 ttl=127 time=2.69 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=4 ttl=127 time=2.53 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=5 ttl=127 time=3.27 ms


Ping de la *VM1* à la *VM2* :

    [root@localhost ~]# ping vm2.tp3.b1
    PING vm2 (192.168.102.10) 56(84) bytes of data.
    64 bytes from vm2 (192.168.102.10): icmp_seq=1 ttl=62 time=2.82 ms
    ^[[A64 bytes from vm2 (192.168.102.10): icmp_seq=2 ttl=62 time=3.09 ms
    64 bytes from vm2 (192.168.102.10): icmp_seq=3 ttl=62 time=3.83 ms
    64 bytes from vm2 (192.168.102.10): icmp_seq=4 ttl=62 time=3.21 ms
    64 bytes from vm2 (192.168.102.10): icmp_seq=5 ttl=62 time=3.54 ms


Ping de la *VM2* au *PC1* :

    [root@localhost ~]$ ping pc1.tp3.b1
    PING pc1 (192.168.112.1) 56(84) bytes of data.
    64 bytes from pc1 (192.168.112.1): icmp_seq=1 ttl=127 time=2.27 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=2 ttl=127 time=2.62 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=3 ttl=127 time=2.43 ms
    64 bytes from pc1 (192.168.112.1): icmp_seq=4 ttl=127 time=2.29 ms


Ping de la *VM2* au *PC2* :

    [root@localhost ~]$ ping pc2.tp3.b1
    PING pc2 (192.168.112.2) 56(84) bytes of data.
    64 bytes from pc2 (192.168.112.2): icmp_seq=1 ttl=127 time=0.356 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=2 ttl=127 time=0.662 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=3 ttl=127 time=0.600 ms
    64 bytes from pc2 (192.168.112.2): icmp_seq=4 ttl=127 time=0.625 ms


Ping de la *VM2* à la *VM1* :

    [root@localhost ~]$ ping vm1.tp3.b1
    PING vm1 (192.168.101.10) 56(84) bytes of data.
    64 bytes from vm1 (192.168.101.10): icmp_seq=1 ttl=62 time=2.50 ms
    64 bytes from vm1 (192.168.101.10): icmp_seq=2 ttl=62 time=2.82 ms
    64 bytes from vm1 (192.168.101.10): icmp_seq=3 ttl=62 time=2.26 ms
    64 bytes from vm1 (192.168.101.10): icmp_seq=4 ttl=62 time=2.53 ms

Tout est OK !!! :)

*Par USEREAU Lucas*