---
layout: default
title: "Proxy Squid"
permalink: /proxy-squid/
---

# Configuration d'un Proxy Squid avec règles de pare-feu et blacklist

## Préambule

Gardez votre infrastructure des séances précédentes. Il faudra juste créer une nouvelle machine cliente avec interface graphique qui sera dans votre LAN (Windows ou Debian).

Si vous avez déjà automatisé votre script iptables via systemd, stoppez-le. Nous allons en créer un autre que l'on exécutera nous-même.

Les buts de cette séance seront :

- Installer un serveur proxy sur la machine Debian qui gère le routage et les règles de sécurité.
- Faire en sorte que le client graphique puisse accéder à Internet et puisse surfer sur des sites (Google, Reddit, Wikipédia, Henallux, ...) tout en restant dans le LAN.
- Configurer le serveur proxy afin que les trafics HTTP et HTTPS passent à travers lui.
- Autoriser le LAN à joindre le proxy explicitement (sans transparence) et bloquer le trafic web direct.
- Créer une blacklist d'URL.

Vérifiez bien que vous avez activé le routage sur votre pare-feu.

## Règles de sécurité iptables

Ajoutez une nouvelle VM avec interface graphique dans votre LAN.

Créez un nouveau script iptables qui pourra :

- Flush les règles des tables Filter et NAT.
- Accepter la connexion SSH de n'importe où vers votre firewall (à but pratique afin que vous puissiez reprendre vos scripts et fichiers de configuration plus facilement).
- Autoriser la connexion tracking (conntrack) pour les nouvelles connexions, établies ou relatives pour le trafic entrant et le trafic traversant le pare-feu.
- Autoriser que le LAN puisse joindre Squid via son numéro de port TCP.
- Autoriser que le LAN puisse interroger des DNS externes (8.8.8.8, ...) et que le trafic DNS passe du LAN vers Internet.
- Refuser que le LAN utilise les ports 80 et 443 pour surfer sur Internet.

De plus, nous allons ajouter une nouvelle règle supplémentaire dans une autre table que Filter.  
Nous allons utiliser brièvement la table NAT afin que le réseau LAN puisse accéder à Internet.  
La règle fera du SNAT et remplacera l'adresse source provenant du LAN par l'adresse IP de l'interface connectée à Internet du pare-feu avant que les paquets ne quittent le device.

On peut utiliser le "Masquerading" si nous avons plusieurs clients dans le LAN.

```bash
iptables -t nat -A POSTROUTING -s $LAN_NET -o $WAN_IF -j MASQUERADE
```

Si vous voulez faire du SNAT statique, il faudra modifier la règle :

```bash
iptables -t nat -A POSTROUTING -s $IP_CLIENT_GRAPH -o $WAN_IF -j SNAT --to-source $IP_IF_FW_INTERNET
```

Après l'exécution de votre nouveau script iptables, vérifiez si vos règles se sont bien appliquées, puis vérifiez via un ping que le LAN puisse communiquer avec `8.8.8.8`.

À ce stade, il est normal que le client graphique ne puisse pas surfer sur Internet puisque vous avez érigé une règle forçant la redirection des trafics HTTP et HTTPS vers Squid (que nous n'avons pas encore configuré).

## Configuration de Squid

Dans le répertoire `/etc/squid`, vous aurez un fichier `squid.conf` déjà présent.  
Je vous conseille de le copier en `squid.sav` et de supprimer le fichier `squid.conf` initial.  
Il est plus rapide de créer nous-même un nouveau fichier de configuration car celui par défaut est très long et complet.

```conf
http_port 3128
visible_hostname firewall-proxy
```

- <span style="color:#e67e22;font-weight:600">(1)</span> Squid va écouter sur le port 3128 pour le trafic web.  
- <span style="color:#e67e22;font-weight:600">(2)</span> On peut lui donner un nom interne que l'on pourra retrouver dans les messages d'erreurs et dans les logs.

```conf
acl localnet src 192.168.2.0/24
```

- <span style="color:#e67e22;font-weight:600">(3)</span> On crée une ACL permettant au réseau précisé d'utiliser le proxy.

```conf
acl SSL_ports port 443
acl Safe_ports port 80 443 1025-65535
acl CONNECT method CONNECT
```

- <span style="color:#e67e22;font-weight:600">(4)</span> On y définit quels ports sont autorisés pour les connexions HTTPS.  
- <span style="color:#e67e22;font-weight:600">(5)</span> On y liste les ports dits "safe", cela évite que le proxy soit utilisé par un attaquant sur d'autres ports (FTP, SMTP, ...).  
- <span style="color:#e67e22;font-weight:600">(6)</span> On précise la méthode CONNECT afin de contrôler la gestion des tunnels HTTPS.

```conf
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
```

> /!\ L’ordre est crucial dans Squid : Squid lit de haut en bas, la première règle qui matche gagne.

- <span style="color:#e67e22;font-weight:600">(7)</span> On refuse tout trafic qui vise un port qui n'est pas Safe_ports.  
- <span style="color:#e67e22;font-weight:600">(8)</span> On refuse toute tentative de CONNECT vers un port autre que le 443.

```conf
acl blacklist dstdomain youtube.com youtu.be ytimg.com googlevideo.com
http_access deny blacklist
```

- <span style="color:#e67e22;font-weight:600">(9)</span> On liste les différents domaines de YouTube (par exemple) dans une ACL nommée blacklist.  
- <span style="color:#e67e22;font-weight:600">(10)</span> On bloque immédiatement toute tentative de requête vers ces domaines.

```conf
http_access allow localnet
http_access deny all
```

- <span style="color:#e67e22;font-weight:600">(11)</span> Tout client venant du LAN précisé en (3) est autorisé si aucune règle précédente ne l'a bloqué.  
- <span style="color:#e67e22;font-weight:600">(12)</span> Tout le reste est refusé.

```conf
access_log /var/log/squid/access.log
cache_log /var/log/squid/cache.log
cache deny all
```

- <span style="color:#e67e22;font-weight:600">(13)</span> Création et gestion du journal des connexions des clients. On y retrouve qui a été sur quel site, ...  
- <span style="color:#e67e22;font-weight:600">(14)</span> Création et gestion du journal du fonctionnement interne de Squid (erreurs, démarrage, ...).  
- <span style="color:#e67e22;font-weight:600">(15)</span> Désactivation du cache pour éviter des surprises liées à du contenu déjà gardé en cache. Si vous n'avez pas utilisé le navigateur jusqu'à maintenant, vous n'êtes pas obligé de l'inscrire dans le fichier de configuration.

### Vérification de la configuration complète

```bash
squid -k parse
systemctl restart squid
systemctl status squid
```

## Configuration du client

Sur Firefox, allez dans les paramètres et cherchez ceux pour la configuration d'un proxy.  
Nous allons le configurer en **manuel** et renseigner l'IP du pare-feu et le port Squid.  
Cochez l'option **Utiliser ce proxy pour tous les protocoles**.

Vous pouvez tester : normalement, si votre blacklist est bonne, le proxy bloquera les sites présents dans la liste.

Créez une deuxième blacklist nommée **social** (domaines de réseaux sociaux) et une troisième nommée **ecole** avec le domaine Henallux.

> Remarque : attention à l'ordre des règles ;)
