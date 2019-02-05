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
    * `net12` et port l'IP `10.0.0.1` et le masque `255.255.0.0`. 
* `router2.tp5.b1` est dans :
    * `net2` et porte l'IP `10.5.2.254/24`
    * `net12` et port l'IP `10.0.0.2` et le masque `255.255.0.0`. 

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
* `net12` : **votre choix** (à justifier)

## Machines

Machine | `net1` | `net2` | `net12`
--- | --- | --- | ---
`client1.tp5.b1` | X | `10.5.2.10` | X
`client2.tp5.b1` | X | `10.5.2.11` | X
`router1.tp5.b1` | `10.5.1.254` | X | *Votre choix*
`router2.tp5.b1` | X | `10.5.2.254` | *Votre choix*
`server1.tp5.b1` | `10.5.1.10` | X | X
