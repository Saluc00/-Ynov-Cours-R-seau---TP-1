TP 2 - Exploration du réseau d'un point de vue client
======================

# Sommaire

* [I. Exploration locale en solo](#i-exploration-locale-en-solo)
    * [Affichage d’informations sur la pile TCP/IP locale](#affichage-dinformations-sur-la-pile-tcpip-locale)
        * [Affichage Gateway](#affichage-gateway)
    * [Affichage configuration réseau GUI](#affichage-configuration-réseau-gui)
    * [Nmap](#nmap)
* [II. Exploration locale en duo](#ii-exploration-locale-en-duo)
    * [Utilisation d’un des deux comme gateway](#utilisation-dun-des-deux-comme-gateway)
    * [Netcat](#netcat)
    * [Wireshark](#wireshark)
* [III. Manipulations d'autres outils/protocoles côté client](#iii-manipulations-dautres-outilsprotocoles-côté-client)
    * [DHCP](#dhcp)
    * [DNS](#dns)
        * [Nslookup](#nslookup)
        * [Reverse lookup](#reverse-lookup)

I. Exploration locale en solo
==========
## Affichage d'informations sur la pile TCP/IP locale

Pour afficher la configuration et l'adressage de nos cartes réseaux on utilise la commande ipcconfig /all 


    Carte réseau sans fil Wi-Fi :
    nom : Killer Wireless-n/a/ac 1535 Wireless Network Adapter
    Adresse MAC : 9C-B6-D0-20-69-A9
    adresse IP : 10.33.3.13
    Broadcast : 10.33.3.255
    Network : 10.33.0.0

    Carte Ethernet Ethernet :
    Nom :  Killer E2500 Gigabit Ethernet Controller
    Adresse MAC : 30-9C-23-18-11-28
    Adresse ip : Aucune 
    Broadcast: Aucun 
    Network: Aucun

    Serveur DHCP 10.33.3.254
    Gateway : 10.33.3.253

## Affichage Gateway

Pour connaitre la gateway il suffit de faire un ipconfig /all 

    Passerelle par défaut: 10.33.3.253

## Affichage configuration réseau GUI

pour afficher les informations réseau d'une carte réseau via l'interface graphique de windows.

* Il faut aller dans le panneau de configuration dans l'onglet réseau et internet
* Onglet centre réseau et partage
* Modifier les parametre de la carte
* Une nouvelle fenetre s'ouvre avec la liste des cartes réseaux du pc

pour accéder aux infos d'une carte réseaux il faut faire un clic droit dessus et choisir Statut et dans cet nouvelle fenetre on à un onglet détails qui nous montrera la config réseau de la carte :

![img](https://github.com/Saluc00/-Ynov-Cours-R-seau---TP-1/blob/master/ressources/Capture2.PNG)

Ou bien :

![img](https://github.com/Saluc00/-Ynov-Cours-R-seau---TP-1/blob/master/ressources/Capture3.PNG)

l'adresse ip MAC et la gateway sont la meme que dans l'exercie en invite de commande ce qui est logique 

La gateway du réseaux ingésup sert à connecter le réseaux LAN ingésup aux réseau WAN et donc de pouvoir accéder à internet il permet aussi de sécuriser son accés et d'interdire certaines requete comme les requetes ICMP pour eviter les DOS et Ddos 

    Premiere adresse IP du réseau 10.33.0.1
    Derniere adresse IP du réseau 10.33.3.254


## Nmap

J'ai installé l'outils `nmap ` *(qui n'ai pas natif dans windows)* Pour scanner le réseau INGESUP. A l'aide de cette outil. je fais la commande `nmap -sn -SE 10.33.0.0/20`.
Apres un certain temps d'attente (160 secondes). Nmap m'affiche toutes les ip, avec adresse mac, nom de machine utilisé sur le réseau.

ensuite il faut faire un nmap du réseau pour vérifier que l'adresse ip que l'on veut prendre n'est pas utilisé.
Malheuresement le firewall du réseau bloque les requetes `ping`.
Pour connaitre une adresse ip qui n'ai pas dans le réseau

* j'ai fais un copié collé du résultat optenue
* Mis le resultat dans un fichier word
* Fais un `ctrl+f`
* taper l'ip `10.33.3.20`

***Pas de resultat...***
Pourquoi ? Car le nmap efféctué précédement ne m'a pas retourné cette ip.
Je peut donc l'utiliser !

Je modifie alors mon adresse ip en `10.33.3.20` car elle était disponible aprés étude du scan 

II. Exploration locale en duo
==========

Pour la seconde parti du TP je me suis connecté en filaire au pc de mon voisin et j'ai changer mon adresse ip en `192.168.0.1`, lui en `192.168.0.2` Puis mis chacun un masque `255.255.255.0`. 

aprés ajout de régle dans le firewall j'arrive à communiquer avec lui et à le pinger

    Envoi d’une requête 'Ping'  192.168.0.2 avec 32 octets de données :
        Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
        Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
        Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128
        Réponse de 192.168.0.2 : octets=32 temps<1ms TTL=128

    Statistiques Ping pour 192.168.0.2:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),

## Utilisation d'un des deux comme gateway

Pour l'exercice suivant mon voisin à servi de passerelle par défaut pour me donner internet pour cela il a fallu qu'il active sa deuxieme carte réseau *(carte wifi connecté au réseau ingesup)* et partager la connection du réseau wifi au pc connecté en filaire.

Pour ce faire :

* Aller dans `modifier les options adaptateur`.
* Configuration de la carte wifi.
* `Propriétés`.
* Aller dans l'onglet `partage`.
* Cocher `Autoriser d'autres utilisateurs du réseau à se connecter via la connexion Internet de cet ordinateur`.

Puis sur le pc qui ne partage pas la connexion wifi, parametrer l'adresse de passerelle avec l'adresse ip du pc qui partage son wifi.

On peut décrire cette étape par ce schema :

```
  Internet           Internet
     X                   |
     X                  WiFi
     |                   |
    PC 1 ---Ethernet--- PC 2
    
- internet joignable en direct par le PC 2
- internet joignable par le PC 1, en passant par le PC 2

```

## Netcat

Pour effectuer un dialogue entre pc dans un meme réseau.

Il nous faut utiliser `Netcat`.

L'un doit etre le serveur et rentrer cela :

`nc.exe -l -p 1048`

![img](https://github.com/Saluc00/-Ynov-Cours-R-seau---TP-1/blob/master/ressources/netcat.png)

*Ce pc créer un serveur de messagerie avec le port 1048*

L'autre doit etre le client et rentrer cela :

`nc.exe 192.168.0.1 1048`

![img](https://github.com/Saluc00/-Ynov-Cours-R-seau---TP-1/blob/master/ressources/netcat%20client.png)

*Ce pc se connecte au serveur de messagerie avec le port 1048*

## Wireshark

J'ai ensuite effectuer une capture de la tram lors de la comunication netcat.

Pour cela :
* Ouvrir Wireshark.
* Rechercher uniquement les trames `TCP`.

On obtient ceci :

![img](https://github.com/Saluc00/-Ynov-Cours-R-seau---TP-1/blob/master/ressources/netcat-tram.png)

III. Manipulations d'autres outils/protocoles côté client
=========

 ## DHCP
 
 Pour afficher le serveur `DHCP` faire :

* Ouvrir un CLI
* `ipconfig /all`

Resultat :
```
Serveur DHCP: 10.33.3.254
```

La date de bail se trouve aussi dans le `ipconfig /all`, soit : 
```
Bail obtenu: jeudi 20 décembre 2018 13:56:59
Bail expiran: jeudi 20 décembre 2018 16:57:49
```

***PETITE DESCRIPTION DU FAMEUX `DHCP`***

Le `DHCP` soit *le protocole de configuration dynamique des hôtes* attribue 'automatiquement' les adresse IP aux machines qui se connecte au réseau.
Ce qui permet de ne pas à avoir a parametrer soit même l'adresse, le masque, la gateway, etc..

Pour changer d'adresse ip en CLI on fait les étapes suivantes : 
* Ouvrir un CLI
* ipconfig /release (***Cette opération supprime notre configuration d'adresse IP***)
* ipconfig /renew (***Cette opération demande au serveur DHCP de nous redonner une configuration d'adresse ip***)

## DNS

En faisant un `ipconfig /all` on peut voir que l'adresse ip du serveur DNS que mon pc connait est : 
```
Serveurs DNS: 10.33.10.20
```
### Nslookup

Enfaisant un `nslookup` dans un CLI j'obtient cela pour `google.com` et `Ynov.com` : 
```
C:\Users\Saluc>nslookup
Serveur par dÚfaut :   UnKnown
Address:  10.33.10.20

> google.com
Serveur :   UnKnown
Address:  10.33.10.20

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:817::200e
          216.58.206.238

> ynov.com
Serveur :   UnKnown
Address:  10.33.10.20

Réponse ne faisant pas autorité :
Nom :    ynov.com
Address:  217.70.184.38
```

On peut voir que le serveur `DNS` de google est `216.58.206.238` et que le serveur `DNS` d'Ynov est `217.70.184.38`. 

### Reverse Lookup

on entre dans le CLI `nslookup 216.58.206.238` pour lui demander quelle est le nom de cette adresse ip.

Avec un `reverse lookup` on obtient cela pour `google.com` : 
```
Nom :    par10s34-in-f14.1e100.net
Address:  216.58.206.238
```

Pour l'ip `78.78.21.21` on obtient : 
```
Nom :    host-78-78-21-21.mobileonline.telia.com
Address:  78.78.21.21
```
et avec cette adresse `96.16.54.88` : 
```
Nom :    host-92-16-54-88.as13285.net
Address:  92.16.54.88
```