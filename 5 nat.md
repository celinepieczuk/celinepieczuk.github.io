# NAT

## Résumé

|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | Mettre en place le NAT par Iptable                           |
| **Objectifs**   | - Configurer SNAT <br />- Configurer DNAT<br />- Ajouter les règles qui autorisent le NAT |
| **Parties**     | - SNAT<br />- DNAT<br />- Exercice dirigé                    |
| **Laboratoire** | -  Ajtouer le DNAT et SNAT au script du firewall             |

## Section I : NAT

### Objectifs de la section

Après cette section, vous serez capable de:

* Créer les règles pour le SNAT avec un IP statique ou dynamique
* Créer les règles pour rediriger du traffic vers un process interne ou un autre hôte grâce à DNAT
* Intégrer ces règles dans un firewall

### Point important pour la configuration du NAT

* En plus de configurer le NAT, n'oubliez de configurer la table filtrer pour autoriser le trafic! 
*  Le NAT traite uniquement le 1er paquet. Les suivants le sont par le connection tracking
*  Vous devrez parfois charger un module pour le support de certains protocoles, comme le FTP, afin que le NAT puisse le supporter.

### SNAT

Le SNAT modifie l'adresse IP source. Cela permet de donner l'accès réseau à des hôtes pour lesquels il n'existe pas de route, comme par ex. un réseau privé derrière un modem-routeur ou dans le cadre de machines virtuelles.

#### IP statique

```bash
iptables -t nat -A POSTROUTING ... SNAT –to-source [<ipaddr>[-<ipaddr>]][:port[-port]]
```

#### Ip dynamique

Ne combinez jamais MASQUERADE et SNAT. 

En cas d'IP statique, privilégiez SNAT.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

#### Exercice dirigé

## DNAT

Le DNAT modifie l'adresse IP de destination.

### DNAT

```bash
iptables -t nat -A PREROUTING -j DNAT --to-destination [<ipaddr>[-<ipaddr>]][:port[-port[/port]]]
```

### REDIRECT

Pour rediriger un trafic réseau vers un processus local, utilisez REDIRECT.

```bash
iptables -t nat -A PREROUTING -j iptables -t nat -A PREROUTING -j REDIRECT --to-ports <port>[-<port>]
```

### Exercice dirigé

Que font les règles suivantes?

```bash
iptables -t nat -A POSTROUTING -o eth0 -j SNAT –to-source 10.1.1.1
iptables -t nat -A POSTROUTING -p tcp -o eth1 -j SNAT --to-source 10.1.1.1-10.1.1.20:1024-32000
iptables -t nat -A POSTROUTING -o eth3 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.1.1-192.168.1.10
iptables -A PREROUTING -t nat -p tcp --dport 80 -i eth4 -s XXXX -j REDIRECT --to-port 3128
```

Remarquez les règles de la table forward nécessaires n'ont pas été précisée!

## Laboratoire

Sur base d'une copie du script des séances précédentes:

* Rendez possible l'accès au Web au clients du lan clients
* Rendez votre serveur web accessible depuis le réseau extérieur (c'est à dire celui de l'école)
* Est-ce que vous pouvez encore accéder aux services de votre voisin? Pourquoi? Si ce n'est plus le cas, corrigez-cela.

## Synthèse

Vous êtes capables d'ajouter le NAT à un firewall Iptables, que ce soit pour pouvoir qu'un réseau "privé" accède au réseau extérieur ou pour rediriger un traffic vers un serveur interne.
