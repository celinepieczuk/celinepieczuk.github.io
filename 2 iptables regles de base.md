---
layout: default
title: "Règles de base"
permalink: /regles-de-base/
---

# Règles de base d'un Firewall

## Résumé

|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | - Découvrir la syntaxe de base d'Iptables et configurer les règles de base d'un firewall. |
| **Objectifs**   | - Découvrir la syntaxe de base d'Iptables. <br />- Configurer les règles minimales d'un firewall. |
| **Parties**     | - Syntaxe de base<br />- Ajouts de règles<br />                |
| **Laboratoire** | - Firewall Iptables de base                                   |

## Section I : Syntaxe de base

### Objectifs de la section

Après cette section, vous serez capable de:

* utiliser la commande Iptables;
* utiliser l'aide.

### Aide

```bash
iptables -h| --help
man iptable
```

### Syntaxe générale

```bash
iptables  [-t table] commande critère action
```

- ​	commande : Tâche à réaliser (ajouter une chaîne, etc.);
- ​	critère : critère de filtrage;
- ​	action : ce que l'on veut faire.

### Visualiser

Lister les règles :

```bash
iptables -L [-t table]
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
....
```

Compteurs de paquets[1](#sdfootnote1sym)

Cela est utile pour optimiser les performances en mettant en premières les règles qui traitent le plus de paquets:

Lister les règles:

```bash
iptables -L -v  --line-numbers -t table [chaîne] 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination   
1        0     0 ACCEPT     udp  --  virbr0 any     anywhere             anywhere             udp dpt:domain


```

* -v : verbeux : ajoute notamment les compteurs pour les chaînes 
* -x : valeur exacte, non arrondie
* --line-numbers : affiche les n° des règles

### Exercice dirigé

Affichez l'aide d'Iptables pour TCP et ICMP

```bash
iptables -p tcp --help
iptables -p -icmp -h
```

## Section II : Ajouts de règles

### Objectifs de la section

Après cette section, vous serez capable de:

* manipuler les règles d'un firewall iptables
* remettre à zéro un firewall
* appliquer une stratégie par défaut

### Introduction

Une règle est une ligne ajoutant un comportement au firewall sur base de critères de filtrage

- Comportement : défini par une cible (target)
  - accept
  - drop
  - reject

- ​	Critères :
  - adresse IP
  - protocole
  - interfaces
  - etc.

### Règles d'Iptables

#### Ajouter une règle

```bash
iptables [-t table] -A chaîne 
```

ex :

```bash
iptables –A INPUT -p icmp -j ACCEPT
```

#### Suppression

```bash
-D chaîne [n° règle | règle]
```

### Remise à zéro d'Iptables

#### Vider une table

```bash
iptables [-t table] -F [chaîne]
```

#### Remise à zéro des compteurs

```bash
iptables -Z chaine
```

### Création d'une règle

#### Ajout d'une cible

```bash
iptables -A <…> -j Target
```

#### Utilisation des critères

```bash
-p protocol
-s adresse source
-d adresse de destination
-i input interface
-o output interface 
```

#### Utilisation des modules

```bash
iptables -m module -...address
```

#### Exemples

Se protéger de l'IP spoofing en bloquant les adresses privées sur une interface "publique".

```bash
iptables -A INPUT -i eth1 -s 192.168.0.0/24 -j DROP
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

Bloquer tout trafic venant du réseau 192.168.0.0/24

```bash
iptables -A INPUT -s 192.168.0.0/24 -j DROP
```

Autoriser en entrée le trafic web non chiffré sur une interface et une adresse IP précise

```bash
iptables -A INPUT -i eth1 -m tcp -p tcp -s 192.168.1.1 --dport 80 -j ACCEPT
```

Logger des paquets avant de les détruire

```bash
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j LOG --log-prefix "IP_SPOOF A: "
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

Accepter sur base d'une adresse MAC

```bash
iptables -A INPUT -m mac --mac-source 00:0F:EA:91:04:08 -j ACCEPT
```

Accepter que les paquets ICMP traversent le firewall 

```bash
iptables -A FORWARD -p icmp  -j ACCEPT
```

Accepter que les paquets ICMP uniquement de type "demande de ping" traversent le firewall

```bash
iptables -A FORWARD -s 192.168.1.0/24 -d 176.16.1.0/16 -i eth0 -o eth1 -p icmp --icmp-type echo-request -j ACCEPT
```

Accepter le trafic entrant des protocoles SSH et Telnet venant du LAN vers le Routeur.

```bash
iptables -A INPUT -s 192.168.1.0/24 -p tcp -m multiport --dport 22,23 -j ACCEPT
```

### Exercice dirigé

A ce niveau, vous pouvez soit travailler directement dans la CLI ou déjà créer des scripts bash.

Listez les règles d'Iptables de manière détaillée et en affichant les n° de port au format numérique. Utilisez une boucle pour afficher les tables filter et nat.

```bash
for i in filter nat raw
do
echo "**********************"
echo "      Table $i "
echo "**********************"

iptables -L -v -n --line-numbers -t $i
done | less
```

Affichez les règles de la chaîne INPUT de la table filter

```bash
iptables -L INPUT
```

Ajoutez une règle qui détruit les paquets venant de 192.168.1.1

```bash
iptables -A INPUT -s 192.168.1.1 -j DROP
iptables -t filter -L INPUT
```


> Remarquez bien que les règles n'autorisent ou ne bloquent le trafic dans un seul sens. Le trafic "retour" n'est pas pris en compte!

Ajoutez quelques règles permettant de passer tout le trafic entre votre LAN et votre Serveur. Utilisez une boucle pour supprimer les règles des tables nat, mangle et filter.

```bash
iptables -A INPUT -s 192.168.1.1 -j DROP
iptables -A INPUT -s 192.168.1.2 -j DROP

iptables -L

for i in nat mangle filter
do
iptables -t $i -F
done

iptables -L
```

## Laboratoire :  réaliser un script de firewall minimal avec l'emploi de variables

### Firewall local avec les règle de bases

Configurez le firewall en respectant les demandes ci-dessous :

1. Démarrage automatique par systemd.
2. Emploi de variables pour les IP's, interfaces, etc.
3. Remise à zéro des tables et flush de toutes les règles.
4. Créez les règles suivantes :
  - Appliquer la règle par défaut.
  - Refuser le ping entre votre Serveur et votre LAN.
  - Autoriser le trafic entrant pour votre interface Loopback
  - Autoriser l'accès à votre site web (HTTP), l'accès SSH et l'accès à votre serveur FTP depuis votre LAN **EN UNE REGLE**.
  - Autoriser l'accès SSH pour accéder à votre Firewall depuis votre PC physique.
6. Le script affiche les règles pour l'ensemble des tables avec les compteurs de paquets.

> Pour ce laboratoire comme pour les suivants, gardez une copie du script pour réviser et une comme base pour le laboratoire de la partie suivante.

## Synthèse

Au cours de ce chapitre, vous avez appris : 

* gérer les règles d'un firewall Iptables
* écrire un script de firewall Iptables qui sera démarré automatiquement.
* Créer les règles de base d'un Firewall.



[1](#sdfootnote1anc)Très utile pour déboguer en vérifiant par quelle règle les paquets passent.

