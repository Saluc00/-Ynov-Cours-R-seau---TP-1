# B1 Réseau 2018 - TP4

*Vous Pourrez retrouver le sujet* [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/4) !
# Sommaire

* [Préparation d'une VM "patron"](#préparation-dune-VM-"patron")
    * [Création des réseaux](#création-des-réseaux)
    * [Mise en place du routage statique](#mise-en-place-du-routage-statique)
* [Spéléologie réseau](#spéléologie-réseau)
    * [ARP](#arp)
        * [Manip 1](#manip-1)
        * [Manip 2](#manip-2)
        * [Manip 3](#manip-3)
        * [Manip 4](#manip-4)

# Préparation d'une VM "patron"

Je créer un *VM* sur VirtualBox :

**Installation et configuration de la VM "patron"** :
* créer une VM
  * 512 Mo RAM
  * 1 CPU
  * Réseau
    * une carte NAT
  * Stockage
    * disque de 8Go
    * `.iso` de CentOS 7
* installation
* wait for installation process to finish
* redémarrer la VM

* configuration VM
  * se logger avec votre utilisateur
  * exécutez :
```
    # Désactivation de SELinux
    sudo setenforce # temporaire
    sudo sed -i 's/enforcing/permissive/g' /etc/selinux/config # permanent

    # Mise à jour des dépôts
    sudo yum update -y

    # Installation de dépôts additionels
    sudo yum install -y epel-release

    # Installation de plusieurs paquets réseau dont on se sert souvent
    sudo yum install -y traceroute bind-utils tcpdump nc

    # Désactivation de la carte NAT au reboot
    sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
    ### mettre ONBOOT à NO

    # Eteindre la machine
    sudo shutdown now
```


## Création des réseaux

Je créez les réseaux suivants :
* le "réseau 1" ou `net1` : `10.1.0.0/24`
  * la carte réseau de l'hôte porte l'IP `10.1.0.1`
* le "réseau 2" ou `net2` : `10.2.0.0/24`
  * la carte réseau de l'hôte porte l'IP `10.2.0.1`

## Création des VMs

Je créé les *VMs* suivantes (= clonez la VM patron !) :
* **VM cliente** ou `client1.tp4`
  * Elle a une carte réseau dans `net1` qui porte l'IP `10.1.0.10`
  * Elle nous servira... de **client**
* **VM serveur** ou `server1.tp4`
  * Elle a une carte réseau dans `net2` qui porte l'IP `10.2.0.10`
  * Elle nous servira de **serveur**
* **VM routeur** ou `router1.tp4`
  * Elle a une carte réseau dans `net1` qui porte l'IP `10.1.0.254`
  * Et une carte réseau dans `net2` qui porte l'IP `10.2.0.254`
  * cette machine sera notre **routeur**. Ce sera la **passerelle** de `client1` et `server1`

En gros ça resemblera a ça :
```
client  <--net1--> router <--net2--> server
```

**Tableau récapitulatif des IP :**

Machine | `net1` | `net2` | @ MAC
--- | --- | --- | --- 
`client1.tp4` | `10.1.0.10` | X | `08:00:27:03:da:f7`
`router1.tp4` | `10.1.0.254` | `10.2.0.254` | `08:00:27:9b:02:7f` *et* `08:00:27:72:37:c4`
`server1.tp4` | X | `10.2.0.10` | `08:00:27:9f:0c:3a`

## Mise en place du routage statique

**SELinux doit être désactivé** (fait dans le patron de *VM* normalement)

**La carte NAT doit être désactivée** (fait dans le patron de *VM* normalement)

On va faire en sorte que notre `client1` puisse joindre `server1`, et vice-versa. 

Pour ce faire : 
* **sur `router1`** : 
    * activer l'IPv4 Forwarding (= transformer la machine en routeur)
      * `sudo sysctl -w net.ipv4.conf.all.forwarding=1`
    * désactiver le firewall (pour éviter certaines actions non voulues)
      * `sudo systemctl stop firewalld` (temporaire)
      * `sudo systemctl disable firewalld` (permanent)
    * vérifier qu'il a déjà des routes pour aller vers `net1` et `net2`
      * `ip route show`

* **sur `client1`** :
    * Mettre le client sur le réseau `10.1.0.0`

* **sur `server1`** :
    * Mettre le client sur le réseau `10.1.0.0`

* **test**
    * `client1` ping `server1` OK !
    * `server1` ping `client1` OK !

Voici le resultat d'un **traceroute** sur le client :

    traceroute to 10.2.0.10 (10.2.0.10), 30 hops max, 60 byte packets
    1  gateway (10.1.0.254)  0.257 ms  0.165 ms  0.263 ms
    2  10.2.0.10 (10.2.0.10)  0.471 ms !X  0.441 ms !X  0.423 ms !X

On se rend alors compte que le client passe par sa **gateway** pour accéder au réseau `10.2.0.0`.  

# Spéléologie réseau

## ARP

Vider la table **ARP** sur toutes les machines Pour chaque manip !

    sudo ip neigh flush all

### Manip 1

* Sur le `client` : 
    * J'affiche la table **ARP** ! Mais rien n'apparait..
    * **EXPLICATION !** Je viens de vider la table, si je l'affiche il n'y a plus rien dedans.

* Sur le `serveur`:
    * J'affiche la table **ARP** ! Mais rien n'apparait..
    * **EXPLICATION !** Je viens de vider la table, si je l'affiche il n'y a plus rien dedans.

* Sur le `client` : 
    * Je ping `serveur`.
    * La table ARP affiche alors : 
    ```
    10.1.0.254 dev enp0s8 lladdr 08:00:27:9b:02:7f DELAY
    ```
    * **EXPLICATION !** Je vois maintenant l'ip que j'ai *route* pour accéder au réseau `10.2.0.0`.

* Sur le `serveur` : 
    * Je ping `client`.
    * La table ARP affiche alors : 
    ```
    10.2.0.254 dev enp0s8 lladdr 08:00:27:27:72:c4 DELAY
    ```
    * **EXPLICATION !** Je vois maintenant l'ip que j'ai *route* pour accéder au réseau `10.1.0.0`.

### Manip 2

* Sur le `router` : 
    * J'affiche la table **ARP** ! Mais rien n'apparait..
    * **EXPLICATION !** Je viens de vider la table, si je l'affiche il n'y a plus rien dedans.

* Sur le `client`:
    * Je ping le `serveur`.

* Sur le `routeur`:
    * Table **ARP** :
    ```
    10.1.0.10 dev enp0s8 lladdr 08:00:27:03:da:7f DELAY
    10.2.0.10 dev enp0s8 lladdr 0a:00:27:9f:c0:3a REACHABLE
    ```

### MANIP 3

* Sur l'`hôte`:
    * Afficher table **ARP** : `arp -a`. (Je la montre pas.. troooop longue !)
    * Supprimer table **ARP** : `arp -d`.
    * Afficher de nouveau la table..
    * J'attend........
    * Afficher encore la table !

Les changement sont que la table c'est agrandit en attendant.
Du fait que, en attendant, des utilisateurs m'ont *certainement* envoyé des paquets..
Apres cela, je les vois dans la table **ARP**.

### MANIP 4
* Sur le `client`:
    * afficher la table ARP
    * activer la carte NAT : `ifup enp0s3` 
    * joindre internet curl `google.com`
    * afficher la table ARP

