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

**But**  
Configurer un pare-feu *stateful* avec `iptables` en s’appuyant sur le *connection tracking* et les différentes formes de NAT.

**Objectifs**

- Comprendre le fonctionnement du connection tracking.
- Comprendre le NAT : SNAT, DNAT, NAT statique, NAT dynamique, REDIRECT.
- Mettre en place des règles `iptables` utilisant le connection tracking et le NAT.
- Écrire un script de firewall stateful réutilisable et maintenable.
- Intégrer le tout dans un environnement local et réseau (LAN1, LAN2, voisins).

---

## 1. Rappels théoriques

### 1.1 Connection tracking : principe

Un firewall *stateless* se contente de regarder chaque paquet individuellement, sans se souvenir du passé.  
Un firewall *stateful* garde en mémoire l’état des connexions. C’est le rôle du **connection tracking** (`conntrack`).

Le *connection tracking* :

- observe les paquets qui traversent le noyau,
- identifie des *flux* (basés sur le fameux 5-tuple : IP source, IP destination, port source, port destination, protocole),
- attribue à chaque flux un état (NEW, ESTABLISHED, RELATED, INVALID),
- met à jour ces états au fil des paquets,
- stocke ces informations dans une table interne au noyau.

#### États principaux

- `NEW`  
  Premier paquet d’une nouvelle connexion (exemple : SYN pour TCP, première requête UDP).

- `ESTABLISHED`  
  Paquet qui appartient à une connexion déjà connue du noyau (flux déjà vu et validé).

- `RELATED`  
  Paquet lié à une connexion existante, mais dans un canal séparé.  
  Exemple typique :  
  - FTP actif ou passif (un flux de données lié au flux de contrôle),  
  - certaines réponses ICMP (erreur liée à une session existante),  
  - protocoles multi-flux (SIP, H.323…).

- `INVALID`  
  Paquet pour lequel le noyau ne parvient pas à déterminer d’état cohérent :  
  - paquet mal formé,  
  - fragment isolé,  
  - paquet arrivant sans qu’on ait vu l’initiation de la connexion, etc.

En pratique, un firewall stateful :

- **autorise** facilement le trafic dans le sens retour (`ESTABLISHED,RELATED`),
- **filtre** principalement le trafic initiateur (`NEW`),
- **rejette** ou loggue les paquets `INVALID`.

#### Exemple d’utilisation dans iptables

```bash
for chain in INPUT OUTPUT FORWARD
do
    # Autorise les paquets de connexions déjà établies ou liées
    iptables -A "$chain" -m state --state ESTABLISHED,RELATED -j ACCEPT
done
```

### Exercice dirigé

Créez un script bash de firewall qui à l'aide de boucles bash:

* remet le fw à zéro pour les tables nat et filter
* active le connection tracking afin d'avoir un firewall statefull

```bash
for i in nat filter
do
iptables -t $i -F
done
for j in INPUT OUTPUT FORWARD
do
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
iptables -A INPUT -s IpDuLan -i CarteDuLan -o CarteAutreLan -m tcp -p tcp --dport 4000:4010 -j ACCEPT 
```

N'oubliez pas de vérifier que tout est bloqué sauf ce qui est autorisé, ici à l'aide de la commande `iptable -L` et de `ping` depuis la loopback et une autre IP.


### 1.2 NAT : principe général

NAT (Network Address Translation) modifie les adresses IP (et éventuellement les ports) des paquets qui traversent un routeur ou un firewall.

Objectifs typiques :

- masquer un réseau privé derrière une adresse publique ;
- publier un serveur interne vers l’extérieur ;
- rediriger ou intercepter certains flux (proxy, filtrage, etc.) ;
- gérer plusieurs réseaux avec des plages d’adresses qui se chevauchent.

Le NAT s’applique dans la table `nat` d’iptables, principalement sur les chaînes :

- `PREROUTING` : avant la décision de routage (pour du DNAT / REDIRECT) ;
- `POSTROUTING` : après la décision de routage (pour du SNAT / MASQUERADE) ;
- `OUTPUT` : pour certains paquets générés localement.

Le NAT fonctionne **avec le connection tracking** :

- le noyau garde une table de correspondance entre les flux originaux et traduits ;
- cela permet de réécrire correctement les paquets dans les deux sens (aller / retour).

### 1.3 SNAT & NAT statique

SNAT (Source NAT) : on modifie l’adresse source du paquet.

Cas typique :  
Un LAN privé sort vers Internet via une seule IP publique. L’IP source privée (ex : `10.0.0.10`) est remplacée par une IP publique (ex : `203.0.113.10`).

Exemple :

```bash
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j SNAT --to-source 203.0.113.10
```
### 1.4 NAT dynamique & MASQUERADE

Dans le NAT dynamique, plusieurs machines d’un réseau privé partagent une ou plusieurs adresses publiques.  
Les ports peuvent être réécrits de manière flexible afin de permettre la coexistence de plusieurs connexions simultanées.

La forme la plus courante de NAT dynamique sous Linux est la cible `MASQUERADE`, qui est une variante de `SNAT`.  
Elle est généralement utilisée lorsque l’adresse IP de l’interface de sortie est **dynamique** (par exemple obtenue via DHCP).

Exemple :

```bash
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
```
### 1.5 DNAT & publication de services

DNAT (Destination NAT) : le DNAT modifie l’adresse de destination (et éventuellement le port) d’un paquet.

C’est typiquement utilisé pour :

- publier un serveur interne sur Internet ;
- rediriger un service vers un autre hôte ;
- faire passer un service situé derrière le firewall pour un service “exposé” sur l’IP publique.

#### Exemple : publication d’un serveur web interne

```bash
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.10:80
```
### 1.6 REDIRECT

**Définition**

- `REDIRECT` est une cible de la table `nat`.
- C’est un cas particulier de DNAT où la destination est modifiée pour pointer vers la machine locale (le firewall lui-même).
- On peut aussi changer le port de destination.

**Utilisation typique**

- Rediriger le trafic HTTP, HTTPS,...
- Intercepter un service pour le filtrer, le journaliser ou l’analyser.
- Forcer l’utilisation d’un service local (proxy, cache, etc.) sans configurer les clients.

**Fonctionnement**

- Le paquet arrive avec :
  - une destination : `IP_firewall:port_original` et
  - un port attendu (ex. 80 pour HTTP).
- La règle `REDIRECT` réécrit la destination vers :
  - `IP_firewall` (l’adresse locale),
  - un autre port (ex. 8080 pour un proxy).
- Le connection tracking assure ensuite le suivi de la connexion et la réécriture correcte des paquets retour.

**Exemple : proxy HTTP transparent**

```bash
# Rediriger tout le trafic HTTP entrant sur eth0 vers un proxy local en 8080
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-ports 8080
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


### Précisions pour le NAT
* En plus de configurer le NAT, n'oubliez pas de configurer la table FILTER pour autoriser le trafic !
* Le NAT traite uniquement le 1er paquet, les paquets suivants seront traités grâce au suivi de connexion (connection tracking).

## 2. Laboratoire :  réaliser un script de firewall statefull

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
