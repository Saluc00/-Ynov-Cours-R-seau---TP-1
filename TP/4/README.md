# B1 Réseau 2018 - TP4

*Vous Pourrez retrouver le sujet* [ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/4) !
# Sommaire

* [Préparation d'une VM "patron"](#préparation-dune-VM-"patron")
    * [Création des réseaux](#création-des-réseaux)
    * [Mise en place du routage statique](#mise-en-place-du-routage-statique)

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

Machine | `net1` | `net2`
--- | --- | ---
`client1.tp4` | `10.1.0.10` | X
`router1.tp4` | `10.1.0.254` | `10.2.0.254`
`server1.tp4` | X | `10.2.0.10` 

## Mise en place du routage statique

**SELinux doit être désactivé** (fait dans le patron de *VM* normalement)

**La carte NAT doit être désactivée** (fait dans le patron de *VM* normalement)

On va faire en sorte que notre `client1` puisse joindre `server1`, et vice-versa. 

Pour ce faire : 
1. **sur `router1`** : 
    * activer l'IPv4 Forwarding (= transformer la machine en routeur)
      * `sudo sysctl -w net.ipv4.conf.all.forwarding=1`
    * désactiver le firewall (pour éviter certaines actions non voulues)
      * `sudo systemctl stop firewalld` (temporaire)
      * `sudo systemctl disable firewalld` (permanent)
    * vérifier qu'il a déjà des routes pour aller vers `net1` et `net2`
      * `ip route show`

2. **sur `client1`** :
    * faire en sorte que la machine ait une route vers `net1` et `net2`
    * laisser le firewall activé

3. **sur `server1`** :
    * faire en sorte que la machine ait une route vers `net1` et `net2`
    * laisser le firewall activé

4. **test**
    * `client1` ping `server1` !
    * `server1` ping `client1` !
    * effectuer un `traceroute` depuis le client pour voir le chemin pris par votre message