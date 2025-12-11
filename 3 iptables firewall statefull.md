---
layout: default
title: "Firewall Statefull + NAT"
permalink: /firewall-statefull/
---


# Firewall statefull + NAT

## Résumé

|                 |                                                              |
| --------------- | ------------------------------------------------------------ |
| **But**         | Configurer un firewall statefull                             |
| **Objectifs**   | - Découvrir la configuration d'un firewall statefull. <br />- Configurer un firewall statefull. |
| **Parties**     | -syntaxe de base<br />- Ajout de règles<br />- Firewall statefull |
| **Laboratoire** | - Firewall local                                             |

## Section I : Firewall statefull

### Objectifs de la section

Après cette section, vous serez capable de:

* Mettre en place un firewall statefull
* Donner un accès à Internet à vos LAN
* Faire une redirection du trafic vers un processus interne avec du DNAT

### Connection tracking

Le module state permet de mettre en place le connection tracking. Une connection peut se trouver dans différents états:

* `NEW` : 1er paquet de la connexion
* `ESTABLISHED` : paquets suivants de la connexion
* `RELATED`: paquets en relation avec la connexion
* `INVALID` : paquets dont l'état n'as pas pu être déterminé

Dans un premier temps, le plus simple est d'autoriser toutes les sessions qui sont déjà établies ou reliée à une session déjà existante. En production, nous pourrions être plus précis. 

```bash
for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
iptables -A $j -m state --state ESTABLISHED,RELATED -j ACCEPT
done
```

### Exercice dirigé

Créez un script bash de firewall qui à l'aide de boucles bash:

* remet le fw à zéro pour les tables nat et filter,
* applique la bonne stratégie par défaut,
* active le connection tracking afin d'avoir un firewall statefull

```bash
for i in nat filter
do
iptables -t $i -F
done
for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
iptables -A $j -m state --state ESTABLISHED,RELATED -j ACCEPT
done
```

Vérifiez que tout est bien bloqué, que ce soit depuis un autre hôte (`ping ...`) ou l'hôte local (`ping 127.0.0.1`)

Modifiez ce script pour qu'il

* N'autorise que l'acceptation des requêtes "ping", le reste du traffic étant bloqué. Les réponse au ping sont autorisée grâce au connexion tracking.
* Accepte le traffic de la boucle locale ("`loopback`") afin que les tests et la communication entre des services locaux puissent fonctionner. Autrement dit le trafic réseau depuis et vers l'adresse `127.0.0.1`
* laisse passer les segment TCP des ports 4000 à 4010 entre les 2 lans

```bash
....

iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -s IpDuLan -i CarteDuLan -d IpAutreLan -o CarteAutreLan -m tcp -p tcp --dport 4000:4010 -j ACCEPT 
```

N'oubliez pas de vérifier que tout est bloqué sauf ce qui est autorisé, ici à l'aide de la commande `iptable -L` et de `ping` depuis la loopback et une autre IP.

### Précisions pour le NAT
* En plus de configurer le NAT, n'oubliez pas de configurer la table FILTER pour autoriser le trafic !
* Le NAT traite uniquement le 1er paquet, les paquets suivants seront traités grâce au suivi de connexion (connection tracking).

### SNAT
Le SNAT modifie l'adresse IP source. Cela permet de donner l'accès réseau à des hôtes pour lesquels il n'existe pas de route.

#### NAT statique

```bash

```
#### NAT dynamique
Ne combinez jamais MASQUERADE et SNAT.

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### DNAT
Le DNAT modifie l'adresse IP de destination.

```bash
iptables -t nat -A PREROUTING -j DNAT --to-destination [<ipaddr>[-<ipaddr>]][:port[-port[/port]]]
```
### REDIRECT

Si vous voulez rediriger un trafic réseau vers un processus local, vous pouvez utiliser REDIRECT.

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


## Laboratoire :  réaliser un script de firewall statefull

### Firewall de type Filtering

A partir du laboratoire précédent, créez un script de firewall qui respecte les demandes ci-dessous.

1. Règles de base pour obtenir un firewall statefull.
2. Démarrage par systemd. Lors de l'arrêt par systemd, les règles sont bien remise à zéro.
3. Variables pour les IP's, interfaces, protocoles, etc.
4. Remise à blanc iptables (R à z table**s**, compteurs, etc.).
5. Autoriser le trafic local (depuis votre IP et la loopback) du fw.
6. Accepter les requêtes DHCP pour le firewall.
8. Autoriser ping vers le fw depuis le lan1.
9. Afficher les règles pour l'ensemble des tables avec les compteurs de paquets.

Faites une copie de sauvegarde du script précédent et modifiez-le en ajoutant :

10. Le routage est pris en charge par le script et non plus par la configuration du réseau. 
11. Lors de l'arrêt par systemd, les routes sont supprimées.
12. Autoriser ping depuis lan1 vers lan2.
13. Autoriser l'administration des machines du lan2 depuis le lan1 (ssh).
14. Autoriser au firewall d'être administréLes par la machine du lan2  .
15. Autoriser l'accès au web (surfer) au lan1 et 2 en un minimum de lignes.

### Firewall Statefull + NAT
16. Faites en sorte d'avoir l'accès à Internet aux clients.
17. Faites en sorte que votre serveur web soit accessible depuis le réseau extérieur (celui de l'école) via le port par défaut.
18. Modifiez la précédente règle afin que votre site web soit disponible via un autre port (non utilisé par un protocole).




[1](#sdfootnote1anc) Ne pas oublier d'autoriser votre firewall a administrer votre voisin de droite !

