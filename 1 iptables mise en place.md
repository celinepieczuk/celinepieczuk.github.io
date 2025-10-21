---
layout: page
title: "Mise en place du laboratoire"
nav_order: 1
---

# Mise en place du laboratoire

## Résumé

|                 |                                                           |
| --------------- | --------------------------------------------------------- |
| **But**         | Préparer le laboratoire                                   |
| **Objectifs**   | - Mettre en place une topologie réseau minimale. <br />   |
| **Parties**     | - Organisation <br />- Rappels <br />- Préparation <br /> |
| **Laboratoire** | - Mise en place de la topologie du laboratoire            |

## Section I : Organisation

### Objectifs de la section

Après cette section, vous serez capable de:

* Consulter les documentations officielles
* vous organiser pour le laboratoire

###  Références

#### Documentation Officielle :

- ​	https://www.netfilter.org/
- ​	http://ipset.netfilter.org/iptables-extensions.man.html
- ​	http://ipset.netfilter.org/iptables.man.html

#### Guides

- ​	Guide complet sur le fonctionnement et la configuration, LA référence : https://www.frozentux.net/documents/iptables-tutorial/
- ​	Guide sur le connection tracking et son utilisée de manière sécurisée https://home.regit.org/netfilter-en/secure-use-of-helpers/
- ​	Iptables et vsftpd https://wiki.archlinux.org/index.php/Very_Secure_FTP_Daemon

#### Chez debian

- ​	Le wiki de debian : debiahttps://wiki.debian.org/iptables
- ​	Le debian handbook : https://debian-handbook.info/browse/stable/sect.firewall-packet-filtering.html

#### Commandes types utiles

- ​	https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

###  Matériel

La manipulation peut se réaliser entièrement en machines virtuelles. Un PC suffit.

Néanmoins il est vivement conseillé de travailler en binômes, surtout pour la manipulation impliquant 2 firewalls.

La préparation des machines virtuelles du laboratoire est à terminer à domicile avant la 2ème séance.

###  Séances

Les explications dans les énoncés sont un résumé de ce qu’il y a dans les références. N’hésitez pas à les consulter !

Pour chaque manipulation :

- ​	**Commentez vos scripts !**
- ​	**Soyez attentif à ne garder aucune règle inutile !**
- ​	***Utiliser les filtres les plus précis possibles (IP, ports, Interfaces,etc.)***
- ​	***Commentez le rôle de chaque ligne !***
- ​	***Vérifiez systématiquement que le trafic réseau :***
  - ​		***bloqué ne peut pas 	passer***
  - ​		***autorisé peut passer***
- ​	***Faites corriger par l'enseignant !***

## Section II : Rappels

### Objectifs de la section

Après cette section, vous serez capable sous Linux de:

* configurer le routage.
* utiliser des variables dans des scripts.
* utiliser Systemd pour créer une unit avec démarrage automatique.

###   Configuration du routage

Ci-dessous un rappel de commandes utilisées pour le routage IPv4.

Le principe est le même pour IPv6 !

```bash
# cat /proc/sys/net/ipv4/ip_forward
1
# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
# cat /etc/sysctl.conf
...
net.ipv4.ip_forward = 0
...

#ip route show
#ip route add 192.168.55.0/24 via 192.168.1.254

# cat /etc/network/interfaces
auto eth0
iface eth0 inet static
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.254
dns-nameservers 10.0.80.11 10.0.80.12
up ip route add 192.168.55.0/24 via 192.168.1.254
down ip route del 192.168.55.0/24 via 192.168.1.254
```

#### Développement d'un calcul IP
```bash
# ``man ipcalc
#`` ipcalc 192.168.0.1/24 
Address:   192.168.0.1          11000000.10101000.00000000. 00000001
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   192.168.0.0/24       11000000.10101000.00000000. 00000000
HostMin:   192.168.0.1          11000000.10101000.00000000. 00000001
HostMax:   192.168.0.254        11000000.10101000.00000000. 11111110
Broadcast: 192.168.0.255        11000000.10101000.00000000. 11111111
Hosts/Net: 254                   Class C, Private Internet
```

### Les variables dans un script bash

```bash
lan1if="eth0"
echo $lan1if
iptables -A INPUT -i $lo -s 127.0.0.1 -j ACCEPT
```

### Le démarrage automatique d'un service par Systemd

La plupart des distributions utilisent systemd pour gérer le démarrage et les services.

L'idéal est de créer 2 scripts : un pour arrêter le firewall (RAZ des règles et stratégie par défaut), l'autre pour charger les règles.

Ensuite, il faut créer la "unit file", par exemple `fw.service` dans le répertoire `/etc/systemd/system/`

```bash
[Unit]
Description=Add Firewall Rules to iptables # description du rôle de la unit

[Service]
Type=oneshot # programme à exécuter pour configurer/ dé configurer et attente de fin d'exécution pour lancer les dépendances 
RemainAfterExit=yes # service vu comme actif si la commande/script est est réussi

ExecStart=/etc/firewall/enable.sh # script de démarrage
#ExecStart=/etc/firewall/enable6.sh  #For IPV6
ExecStop=/etc/firewall/stop.sh # script de démarrage
#ExecStop=/etc/firewall/stop6.sh  #For IPV6

[Install]
WantedBy=multi-user.target # dans quel contexte démarrer le service 
```

Lancer, arrêter, etc, le service se fait via la commande habituelle, `systemctl `: 

```bash
systemctl enable fw.service
```



## Section III : laboratoire, préparation de la topologie

### Objectifs de la section

Après cette section, vous serez capable sous Linux de:

* mettre en place une topologie de base avec routeurs, lans, clients, serveurs.

###  Topologie à préparer

#### Schéma

PC1 et PC2 sont les ordinateurs respectifs d’un binôme d’étudiants. Les topologies sur PC1 et PC2 sont identiques et sont réalisées à l’aide de VMs.

Dans chaque binôme, un étudiant réalise l’ensemble des exercices sur la topologie du PC1, tandis que l’autre que sur la topologie du PC2.

Les exercices sont individuels sauf lorsqu’il faut communiquer avec les réseaux LAN d’un voisin de gauche ou droite.

![Schéma du labo]({{ '/assets/images/schema-labo.png' | absolute_url }})

#### Réseaux

X=1 pour les VM de PC1

X=2 pour les VMS de PC2

- Lan1 : Lan des « clients »
  - 172.16.X.0/24
  - Client  linux ou autre
  - ip dynamique dhcp reçue du routeur/FW
  - réseaux interne intnet1
- Lan2 : Lan des « serveurs »
  - 192.168.X.0/24
  - Serveurs Linux
  - ip statique 
  - réseaux interne intnet2
- Lan entre PC1 et PC2 
  - Adressage réseau du local

#### Client

- Iface
  - dhcp
  - intnet1


#### Services

- VM linux Firewall
  - 3 interfaces
    - Type bridge  10.1.31.X(Mettre en statique l'IP reçue par le DHCP) vers le réseau extérieur
    - Réseau interne  		
      - Intnet1 pour le lan1 (172.16.X.254/24)
      - Intnet2 pour le lan2 (192.168.X.254/24)
  - services
    - Proxy
    - Dhcp + routeur
    - SSH
- ​	Serveur
  - iface
    - Réseau intnet2
    - 192.168.X.1/24
  - services
    - WWW
    - FTP
    - SSH

### Mise en pratique

1. Installez et configurez les services sur le serveur et le routeur.
2. Réalisez une version papier de la topologie avec les IP, interfaces, physiques et virtuelles, etc.
3. Activez en place le routage entre les réseaux du client et du serveur.
4. Configurez correcetement le réseau afin que les pare feux accèdent à Internet.
4. Configurez les routes statiques.Vos réseaux doivent pouvoir communiquer avec les réseaux de votre voisin. Les routes statiques seront gérée plus tard par le script du firewall.

## Synthèse

Au cours de ce chapitre, vous avez appris : 

* vous organiser pour ce cours
* mettre à profit les concepts comme le routage, les variables, systemd pour préparer un système Linux à être un firewall
* configurer une topologie réseau minimale 

