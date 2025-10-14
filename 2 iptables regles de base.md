---
title: "Règles de base d'un Firewall"
layout: default
---

# Règles de base d'un Firewall

## Résumé
...


|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | Découvrir la syntaxe de base d'Iptable et configurer les règles de base d'un firewall |
| **Objectifs**   | - Découvrir la syntaxe de base d'Iptables. <br />- Configurer les règles minimales d'un firewall. |
| **Parties**     | -syntaxe de base<br />- Ajout de règles<br />                |
| **Laboratoire** | - Firewall Iptable de base                                   |

## Section I : syntaxe de base

### Objectifs de la section

Après cette section, vous serez capable de:

* utiliser la commande Iptable;
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

Aussi pour optimiser les performances en mettant en 1er les règles qui traitent le plus de paquets:

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

## Section II : ajouts de règles

### Objectifs de la section

Après cette section, vous serez capable de:

* manipuler les règles d'un firewall iptables
* remettre à zéro un firewall
* appliquer une stratégie par défaut

### Introduction

Une règle est une ligne ajoutant un comportement au firewall sur base de critères de filtrage

- Comportement : défini par une cible (target)
  - log
  - drop
  - etc.
- ​	Critères :
  - adresse IP
  - protocole
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

#### Remplacement

```bash
-R chaine n° logrègle (Cfr –line-numbers)
```

ex :

```bash
iptables –R 2 INPUT -p icmp --icmp-type echo-reply  -j ACCEPT 
```

#### Suppression

```bash
-D chaîne [n° règle | règle]
```

#### Insertion

```bash
-I chaîne [num] règle (pas num = 1 sous entendu)
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

### Stratégie par défaut

```bash
iptables -t table -P chaîne action (action : ACCEPT; DROP)
ex :
iptables -P chain DROP
```

### Création d'une règle

#### ajout d'une cibles

```bash
iptables -A <…> -j Target
```

#### Utilisation des critères

```bash
-p protocol
-s source address
-d destination address
-i input interface
-o output interface 
-f fragmented
```

#### utilisation des modules

```bash
iptables -m module -...address
```

#### Exemples

Se protéger de l'IP spoofing en bloquant les adresses privées sur une interface "publique".

```bash
iptables -A INPUT -i eth1 -s 192.168.0.0/24 -j DROP
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

Bloquer un réseau

```bash
iptables -A INPUT -s 192.168.0.0/24 -j DROP
```

autoriser en entrée le traffic web non chiffré sur une interface et une adresse IP précise

```bash
iptables -A INPUT -i eth1 -m tcp -p tcp -s 192.168.1.1 --dport 80 -j ACCEPT
```

Logger des paquet avant de les détruire

```bash
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j LOG --log-prefix "IP_SPOOF A: "
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP
```

Accepter sur base d'une adresse mac

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

### Exercice dirigé

Vous pouvez soit travailler directement dans la CLI ou créer des scripts bash.

Listez les règles d'Iptables de manière détaillée et en affichant les n° de port au format numérique et les n° des règles. Utilisez une boucle pour afficher les tables filter et nat.

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

Insérez une règle après la règle une

```bash
iptables -I INPUT 2 -s 192.168.1.2 -j DROP
iptables -L INPUT -n --line-numbers
```

> Remarquez bien que les règles n'autorisent ou ne bloquent le trafic dans un seul sens. Le trafic "retour" n'est pas pris en compte!

Supprimez la 1ere règle sur base de son numéro. La n°2 devient la n°1. Supprimez la sur base de son "contenu"

```bash
iptables -L --line-numbers
iptables -D INPUT 1
iptables -L --line-numbers
iptables -D INPUT -s 192.168.1.2 -j DROP
iptables -L --line-numbers
```

A l'aide d'une boucle, choisissez la bonne stratégie par défaut

```bash
for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
done
```

Ajoutez quelques règles. Utilisez une boucle pour supprimer les règle des tables nat, mangle et filter.

```bash
iptables -A INPUT -s 192.168.1.1 -j DROP
iptables -I INPUT 2 -s 192.168.1.2 -j DROP

iptables -L

for i in nat mangle filter
do
iptables -t $i -F
done

iptables -L
```

## Laboratoire :  réaliser un script de firewall minimal avec emploi de variables

### Firewall local avec les règle de bases

Configurez le firewall en respectant les demandes ci-dessous :

1. Démarrage automatique par systemd.
2. Emploi de variables pour les IP's, interfaces, etc.
3. Remet à blanc iptables (R à z tables et compteurs).
4. Le script affiche les règles pour l'ensemble des tables avec les compteurs de paquets.

> Pour ce laboratoire comme pour les suivants, gardez une copie du script pour réviser et une comme base pour le laboratoire de la partie suivante.

## Synthèse

Au cours de ce chapitre, vous avez appris : 

* gérer les règles d'un firewall Iptables
* écrire un script de firewall Iptables qui sera démarré automatiquement.
* Créer les règles d'un Firewall 



[1](#sdfootnote1anc)Très utile pour déboguer en vérifiant par quelle règle les paquets passent.

