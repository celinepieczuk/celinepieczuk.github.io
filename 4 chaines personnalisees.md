---
layout: page
title: "Chaînes personnalisées"
nav_order: 4
permalink: /chaines-personnalisees/
---

# Chaînes personnalisées

## Résumé

|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | Personnaliser les chaînes d'un firewall Iptable              |
| **Objectifs**   | - Découvrir les chaînes personnalisées <br />- Mettre en place des chaînes personnalisée |
| **Parties**     | - Intérêt<br />- Création et utilisation<br />- Exercice dirigé |
| **Laboratoire** | - création et utilisation d'un chaîne personnalisée          |

## Section I : Chaînes personnalisées

### Objectifs de la section

Après cette section, vous serez capable de:

* Expliquer l'utilité d'une chaîne personnalisée
* Créer et utiliser une chaîne personnalisée

### Introduction

Vous êtes souvent confronté à devoir mettre en place le même élément plusieurs fois. Que ce soit dans une configuration, un logiciel, etc. Grâces aux templates, fonctions, objets, etc. vous évitez ces répétitions. En évitant ces copier-coller, vous limitez le risques d'erreurs, d'oubli de mise à jour, etc.

Les chaînes personnalisée sont un peu l'équivalent des fonctions dans Iptables et permettent donc d'utiliser un jeu de règles plusieurs fois.

### Création d'une chaîne personnalisée

#### Conventions :

* minuscules -> une chaîne personnalisée

* MAJUSCULES ->  chaîne prédéfinie

#### Ajout :

```bash
iptables -N chaîne
```

#### Suppression :

```bash
iptables -X chaîne
```

#### renommer :

```bash
iptables -E chaîne nouveauNom
```

### Exercice dirigé

Créez une chaîne personnalisée nommée "privateNets".

```bash
iptables -N privateNets
```

Ajoutez les règles pour filtrer les réseaux privés. Remarquez bien les masques choisis!

```bash
iptables -A privateNets -s 192.168.0.0/16 -j DROP
iptables -A privateNets -s 10.0.0.0/8 -j DROP
iptables -A privateNets -s 172.16.0.0/12 -j DROP
```

Vérifiez le contenu de votre chaîne.

iptables -L privateNets

Utilisez votre chaîne pour tout le trafic qui traversera votre firewall. Vérifiez.

```bash
iptables -t filter -A FORWARD -j privateNets
```

### Laboratoire

Certaines combinaisons de flag TCP ne sont pas censées exister. Par exemple tous les flags à 1. Il y en a bien entendu d'autres.

Reprenez le firewall des séances précédentes et créez une chaîne personnalisée qui droppe les segments TCP avec tous les flags à 1. Appliquez cette chaîne à tout les trafic réseaux (astuce : utilisez une boucle).

## Synthèse

Au cours de ce chapitre, vous avez appris à créer et à utiliser des chaînes personnalisée Iptables, tout en revoyant comment filtrer sur base des drapeaux TCPs.
