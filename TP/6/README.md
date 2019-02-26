# B1 Réseau 2018 - TP6

[Ici](https://github.com/It4lik/B1-Reseau-2018/tree/master/tp/6) l'énoncé du TP 6..

# Sommaire

* [Matériels utilisés](#matériels-utilisés)
* [Présentation du lab](#présentation-du-lab)
    * [Aires OSPF](#aires-ospf)
    * [Réseaux IP et aires OSPF](#réseaux-ip-et-aires-ospf)
    * [Adressage IP de chacune des machines](#adressage-ip-de-chacune-des-machines)

# Matériels utilisés

* Logiciel utilisés:
    * Virtualbox
    * GNS3

* les machines virtuelles Linux :
    * CentOS 7

* les routeurs:
    * Cisco 3640

* les switches :
    * Ethernet Switch dans GNS3

# Présentation du lab

Petit schema récapitulatif :

<p align="center">
  <img src="./photo/topo.png" title="Topologie du Lab 2">
</p>

(Oui j'ai piqué la photo :P )

## Aires OSPF

Dans se TP je decouvre **OSPF**.

Ou je met 3 *area* en place.

Nous allons nous servir des aires OSPF pour distinguer :

* Une aire "backbone" *(l'aire principale)*
    
        AREA 0
    
    * toutes les autres y sont connectées
        donc tout trafic qui change d'aire passe forcément par celle-ci

    * le **WAN** (~= internet) est souvent accessible depuis l'aire "backbone"

* Une aire pour les clients
    
        AREA 1
    
    * X clients seront ajoutés (Quelque chose comme 2/3 clients)

    * leur adressage IP et leur table de routage seront gérées automatiquement

    * Un serveur DHCP 

* Une aire pour les services d'infrastructures

        AREA 2

    * Un serveur web ou un netcat simple, pour simuler un service un service disponible

## Réseaux IP et aires OSPF

Dans chacune des aires OSPF se trouveront des réseaux IP.

Réseaux | `area 0` | `area 1` | `area 2` | Infos
--- | --- | --- | --- | ---
`10.6.100.0/30` | X | - | - | Liaison entre `r1` et `r2`
`10.6.100.4/30` | X | - | - | Liaison entre `r1` et `r4`
`10.6.100.8/30` | X | - | - | Liaison entre `r2` et `r3` 
`10.6.100.12/30` | X | - | - | Liaison entre `r3` et `r4`
`10.6.101.0/30` | - | X | - | Liaison entre `r3` et `r5`
`10.6.201.0/24` | - | X | - | Réseau des clients
`10.6.202.0/24` | - | - | X | Réseau des serveurs

## Adressage IP de chacune des machines

Machines | `10.6.100.0/30` | `10.6.100.4/30` | `10.6.100.8/30` | `10.6.100.12/30` | `10.6.101.0/30` | `10.6.201.0/24` | `10.6.202.0/24`
--- | --- | --- | --- | --- | --- | --- | --- 
`r1.tp6.b1` | `10.6.100.1` | `10.6.100.5` | - | - | - | - | `10.6.202.254`
`r2.tp6.b1` | `10.6.100.2` | - |  `10.6.100.9` | - | - | - | -
`r3.tp6.b1` | - | - | `10.6.100.10` | `10.6.100.14` | `10.6.101.1` | - | -
`r4.tp6.b1` | - |  `10.6.100.6` | - | `10.6.100.13` | - | - | -
`r5.tp6.b1` | - | - | - | - |  `10.6.101.2` |  `10.6.201.254` | -
`client1.tp6.b1` | - | - | - | - | - |  `10.6.201.10` | -
`client2.tp6.b1` | - | - | - | - | - |  `10.6.201.11` | -
`server1.tp6.b1` | - | - | - | - | - | - | `10.6.202.10`