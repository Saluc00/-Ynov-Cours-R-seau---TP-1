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
    * [Wireshark](#wireshark)
* [Interception d'ARP et ping](#Interception-dARP-et-ping)
* [Interception d'une communication netcat](#Interception-dune-communication-netcat)

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
    ```
    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
    ```
Cette *IP* est l'ip de la passerelle du réseau sur laquelle est la carte *NAT*.

## Wireshark

Go installer **tcpdump** !

    yum install tcpdump

*Surtout, ne pas oublier d'avoir la carte NAT (du router1) d'allumé !*

je me connecte au ***SSH*** de mon router à l'adresse `10.1.0.254`.

Sur **Powershell** faire :

    ssh root@10.1.0.254

*mettre le mdp du router1*

#### Maintenant..

Vu que je suis connecté en SSH, j'envoye des trames en permanence sur l'IP `10.1.0.254`.
Je vais donc capturer le trafic de l'interface à laquelle je ne suis **PAAAAS** connecté pour éviter le bruit généré par **SSH**.

# Interception d'ARP et ping



- sur router1:

    Lancer Wireshark pour enregistrer le trafic qui passer par l'interface choisie et enregistrer le trafic dans un fichier ping.pcap :

        sudo tcpdump -i enp0s9 -w ping.pcap

Ok ! le router *écoute* les trames !

- sur client1: 

    * vider la table ARP
    * envoyer 4 pings à `10.2.0.10`.

        ping -c 4 10.2.0.10`

- sur router1: 

    * quitter la capture (CTRL + C)
    * vérifier la présenc edu fichier ping.pcap avec un ls
    * envoyer le fichier ping.pcap sur votre hôte 

*Pour récuperer le fichier `ping.pcap`*

**Para recuperar el archivo:**

*(Pour récuperer le fichier)*
* A l'aide de `fileZilla`.
    * Inserer :
        * Ip de déstination.
        * identifient.
        * Mot de passe.
        * GOGOGOGO

Puis télécharger le fichier !!! (ping.pcap)

- sur l'hôte (votre PC) :

    * ouvrir le fichier ping.pcap dans Wireshark
    * J'y vois :
        * La question pour connaître la MAC de la destination.
            ```
            Who has 10.1.0.254? Tell 10.2.0.10
            ```
        * La réponse :
            ```
            10.1.0.254 is at 08:00:27:72:37:c4
            ```
        * Puis les **ping** !
        * Puis les **pong** !

# Interception d'une communication `netcat`

**Intercepter le trafic**

* Depuis router1
    * Pendant que client1 se connecte au serveur netcat de server1
        * Pour se connecter :
            * Server1 : `nc -l 9999` (Pour le port 9999) !
            * Client1 : `nc 10.2.0.10 9999` !
    
Recuperer le fichier avec FileZilla

* le client envoie SYN : demande de synchronisation
    *  Soit : `[SYN] Seq=0 Win=29200 Len=0 MSS=1460 SACK_PERM=1 TSval=8671283 TSecr=0 WS=128`
* le serveur répond SYN,ACK : il accepte la synchronisation
    * Soit : `[SYN, ACK] Seq=0 Ack=1 Win=28960 Len=0 MSS=1460 SACK_PERM=1 TSval=8671823 TSecr=8671283 WS=128`
* le client répond ACK : Envoie de la confirmation de la connection
    * `[ACK] Seq=1 Ack=1 Win=29312 Len=0 TSval=8671285 TSecr=8671823`

**Remettre le firewall !**



