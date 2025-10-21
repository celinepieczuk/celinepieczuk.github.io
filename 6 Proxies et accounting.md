---
layout: default
title: "Proxies et Accounting"
permalink: /proxies-accounting/
---


# Chaînes personnalisées

## Résumé

|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | Compter les paquets et configurer un proxy                   |
| **Objectifs**   | -  Configuration de base de SQUID <br />- Configuration d'un proxy obligatoire <br />- Configuration d'un proxy transparent<br />- Compter les paquets selon différents critères |
| **Parties**     | - Proxies<br />- Accounting<br />                            |
| **Laboratoire** | - Proxy transparent et obligatoire<br />- IP Accouting       |

## Section I : Proxies

### Objectifs de la section

Après cette section, vous serez capable de:

* Mettre un place un configuration minimum de SQUID
* Ajouter les règles de firewall pour un proxy obligatoire ou transparent

### Configuration de SQUID

Le fichier de configuration de SQUID est très long et bien commenté. Comme pour n'importe quel fichier de configuration, faites une copie de sauvagarde afin de pouvoir revenir en arrière si nécessaire.

Par défaut, SQUID écoute sur le port `3128`

Au minimum, vous  ajouterez une ACL qui sera utilisée pour préciser les réseaux pour lesquels SQUID traitera les requêtes.

Passez en revue rapidement le fichier de configuration.

Configuration minimum :

```bash
/etc/squid.conf

acl our_networks src 10.1.6.0/24 192.168.2.0/24

http_access allow our_networks
http_access allow localhost

[http_access deny all]
```

### Proxy obligatoire

Pour rendre un proxy obligatoire, bloquez l'accès vers l'extérieur au trafic réseau concerné, à l'exception du proxy. L'utilisation du proxy est désormais obligatoire.

### Proxy transparent

Pour rendre un proxy transparent, le trafic sera en plus d'être bloqué vers l'extérieur redirigé par du NAT vers le démon proxy.

### Exercice dirigé



## Laboratoire :  

* Revenez à une config firewall de base

* Configurez un proxy obligatoire

  * Le proxy est situé sur le firewall
  * Le pc client doit passer obligatoirement par un proxy pour accéder au WEB
    * Seul le proxy à la droit de sortir en http et https
    * Les hôtes n'ont accès en http et https qu'au proxy

* Convertissez-le en proxy transparent

  * Seuls les paquets http sont redirigés (NAT) vers le proxy
  * Les hôtes ne sont pas configurés pour utiliser un proxy

NB : Dans le cas d'un proxy obligatoire, le proxy doit pouvoir effectuer des requêtes DNS. Dans le cas d'un proxy transparent, le client doit pouvoir effectuer des requêtes DNS.

## Section II IP accounting (pour aller plus loin)

### Objectifs de la section

Après cette section, vous serez capable de:

* comptabiliser les paquets qui traversent un hôtes sur base de certains critères.

### Configuration

La mise en place est assez simple : il faut créer des règles ne font "rien", c'est à dire sans cibles.

```bash
iptables -A FORWARD -j eth0_OUT
```

La création de chaînes personnalisées permet de regrouper les règles pour regrouper des critères de comptage

```bash
iptables -N eth0_OUT
```

Il ne restera plus qu'à étudier les résultats de ces compteurs :

```bash
Iptables -v -n -L NomChainePersonnalisee
```

### Laboratoire

Remettez à 0 les règles du FW. Tout le trafic peut passer, mais il fait du Nat pour le lan1 comme précédemment et fait de l'accounting :

* Compter le trafic traversant le FW venant du LAN1

* Compter séparément le trafic via une chaîne personnalisée qui regroupe :

  * DNS

  - Http

  - Https

  - Vers le portail henallux

* Grâce à la commande watch, afficher en temps réel le trafic

## Synthèse

Au cours de ce chapitre, vous avez appris:

* Configurer un proxy SQUID minimal avec les règles Iptables associées
* Mettre en place un IP accounting minimal avec Iptable
