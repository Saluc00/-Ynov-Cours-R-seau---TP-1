# TP 3 - Plusieurs réseaux : routage statique

*Vous pourrez retrouver le sujet* [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/3) !

# Sommaire

* [Matériel utilisé](#matériel-utilisé)
* [Initiation](#initiation)

# Matériel utilisé

Pour se *TP* j'utilise une **machine virtuelle** 

**Logiciel** : `VirtualBox`
**OS** : `centOS 7 (minimal)`

# Initiation

Dans notre TP, j'utilise **une machine virtuelle** (*VM*) avec un *OS* [centOS](https://fr.wikipedia.org/wiki/CentOS).

### Avant de commencer

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

Petit `ifdown ens37` et `ifup ens37` pour redemarrer la carte réseau *(Pour etre sur que la config sois correctement mise)*

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


Pour ***prouver*** que j'ai bien un liaison entre mon pc et la *VM*.
Je fais un **ping** depuis mon pc vers la *VM* avec la commande `ping 192.168.126.10`.

    ping 192.168.126.10

    Envoi d’une requête 'Ping'  192.168.126.10 avec 32 octets de données :
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.126.10 : octets=32 temps<1ms TTL=64

    Statistiques Ping pour 192.168.126.10:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
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

*Bien sur avant de faire cette commande, la configuration de la carte réseau host-only est* `192.168.126.1` *avec un masque de sous réseau* `255.255.255.0`

