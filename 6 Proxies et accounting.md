---
layout: default
title: "Proxy Squid"
permalink: /proxy-squid/
---

# Configuration d'un Proxy Squid avec Règles de Pare-feu et Blacklist

## Préambule
Gardez votre infrastructure des séances précédentes. Il faudra juste créer une nouvelle machine Cliente avec interface graphique qui sera dans votre LAN (Windows ou Debian)

Si vous avez déjà automatisé votre script iptables via systemd, stoppez-le. Nous allons en créer un autre que l'on exécutera nous-même. 

Les buts de cette séance seront :

- Installer un serveur proxy sur la machine Debian qui gère le routage et les règles de sécurité.
- Faire en sorte que le client graphique puisse accéder à internet et puisse surfer sur des sites (Google, Reddit, Wikipédia, Henallux,...) tout en restant dans le LAN.
- Configurer le serveur proxy afin que les trafics HTTP et HTTPS passent à travers lui.
- Autoriser le LAN à joindre le proxy explicitement (sans transparence) et bloquer le trafic web direct.
- Créer une blacklist d'URL.


Vérifiez bien que vous avez activé le routage sur votre Pare-feu.

## Règles de sécurité Iptables
Ajoutez une nouvelle VM avec interface graphique dans votre LAN.

Créez un nouveau script IPtables qui pourra:
- Flush les règles des tables Filter et NAT
- Accepter la connexion SSH de n'importe où vers votre Firewall (à but pratique afin que vous puissiez reprendre vos scripts et fichiers de configuration plus facilement)
- Autoriser la connexion tracking (conntrack) pour les nouvelles connexions, établies ou relatives pour le trafic entrant et le trafic traversant le pare-feu
- Autoriser que le LAN puisse joindre Squid via son numéro de port tcp
- Autoriser que le LAN puisse interroger des DNS externes (8.8.8.8,...) et que le trafic DNS passe du LAN vers Internet
- Refuser à ce que le LAN utilise les ports 80 et 443 pour surfer sur internet


De plus, nous allons ajouter une nouvelle règle supplémentaire dans une autre table que Filter. Nous allons utiliser brièvement la table NAT afin que le réseau LAN puisse accéder à Internet. La règle fera du SNAT et fera en sorte que remplacer l'adresse source provenant du LAN par l'adresse IP de l'interface connectée à Internet du pare-feu avant qu'ils ne quittent le device. On peut utiliser le "Masquerading" si nous avons plusieurs clients dans le LAN.

```bash
iptables -t nat -A POSTROUTING -s $LAN_NET -o $WAN_IF -j MASQUERADE
```
Si vous voulez faire du SNAT statique, il faudra modifier la règle.
```bash
iptables -t nat -A POSTROUTING -s $IP_CLIENT_GRAPH -o $WAN_IF -j SNAT --to-source $IP_IF_FW_INTERNET
```

Après l'exécution de votre nouveau script IPtables, vérifier si vos règles se sont bien appliquées et vérifiez via un ping que le LAN puisse communiquer avec 8.8.8.8.

A ce stade, il est normal que le client graphique ne puisse pas surfer sur Internet puisse que vous avez érigé une règle forçant la redirection des trafics HTTP et HTTPS vers le Squid (que nous n'avons pas encore configuré)


## Configuration de Squid

Dans le répertoire /etc/squid, vous aurez un fichier squid.conf déjà présent. Je vous conseille de le copier squid.sav et de supprimer le fichier squif.conf initial. Il est plus rapide de créer nous-même un nouveau fichier de configuration car celui par défaut est très long et complet. 
```bash
http_port 3128 (1)
visible_hostname firewall-proxy (2)
```
(1)Squid va écouter sur le port 3128 pour le trafic web. (2) On peut lui donner un nom interne que l'on pourra retrouver dans les messages d'erreurs et dans les logs.
```bash
acl localnet src 192.168.2.0/24 (3)
```
(3)On crée une ACL permettant au réseau précisé d'utiliser le proxy.
```bash
acl SSL_ports port 443 (4)
acl Safe_ports port 80 443 1025-65535 (5)
acl CONNECT method CONNECT (6)
```
(4) On y définit quels ports sont autorisés pour les connexions HTTPS
(5) On y liste les ports dit 'safe', cela évite que le proxy soit utilisé par un attaquant sur d'autres ports (FTP,SMTP,...)
(6) On précise la méthode CONNECT afin de contrôler la gestion des tunnels HTTPS
```bash
http_access deny !Safe_ports (7)
http_access deny CONNECT !SSL_ports (8)
```
/!\ L’ordre est crucial dans Squid : Squid lit de haut en bas, la première règle qui matche gagne.
(7) On refuse tout trafic qui vise un port qui n'est pas Safe_ports
(8) On refuse toute tentative de CONNECT vers un port autre que le 443
```bash
acl blacklist dstdomain youtube.com youtu.be ytimg.com googlevideo.com (9)
http_access deny blacklist (10)
```
(9) On liste les différents domaines de YouTube (par exemple) dans une ACL nommée blacklist
(10) On bloque immédiatement toute tentative de requête vers ces domaines
```bash
http_access allow localnet (11)
http_access deny all (12)
```
(11) Tout client venant du LAN précisé en (3) est autorisé si aucune règle précédente ne l'a bloqué
(12) Tout le reste est refusé
```bash
access_log /var/log/squid/access.log (13)
cache_log /var/log/squid/cache.log (14)
cache deny all (15)
```
(13) Création et gestion du journal des connexions des clients. On y retrouve qui a été sur quel site,...
(14) Création et gestion du journal du fonctionnement interne de Squid (erreurs, démarrage,...)
(15) Désactivation du cache pour éviter des surprises liées à du contenu déjà gardés en cache. Si vous n'avez pas utilisé le navigateur jusqu'à maintenant, vous n'êtes pas obligé de l'inscrire dans le fichier de configuration


### Vérification de la configuration complète
```bash
	squid -k parse
	systemctl restart squid
	systemctl status squid
```
## Configuration du Client

Sur Firefox, allez dans les paramètres et chercher ceux pour la configuration d'un Proxy. Nous allons le configurer en "manuel" et renseignez lui l'IP du pare-feu et le port Squid. Cocher l'option **Utiliser ce proxy pour tous les protocoles**.

Vous pouvez tester et normalement si votre blacklist est bonne, le proxy bloquera les sites présents dans la liste.

Créez une deuxième blacklist nommée **social** avec d'autres domaines relatifs aux réseaux sociaux et une troisième nommée **ecole** avec le domaine Henallux.

Remarque: Attention à l'ordre des règles ;) 
