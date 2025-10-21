# Organisation

##  Références

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

##  Matériel

La manipulation peut se réaliser entièrement en machines virtuelles. Un PC suffit.

Néanmoins il est vivement conseillé de travailler en binômes pour la partie en P.  14 : Manipulation : règles supplémentaires concernant du trafic entrant et sortant de réseaux extérieurs et des services supplémentaires.

La préparation des machines virtuelles du laboratoire est à terminer à domicile après la 1ere séance.

##  Séances

Les explications dans les énoncés sont un résumé de ce qu’il y a dans les références. N’hésitez pas à les consulter !

Pour chaque manipulation :

- ​	**Commentez vos scripts !**
- ​	**Soyez attentif à ne garder aucune règle inutile !**
- ​	***Utiliser les filtres les plus précis possibles (IP, ports, Interfaces,etc.)***
- ​	***Commentez le rôle de chaque ligne !***
- ​	***Vérifiez systématiquement que le trafique réseau***
  - ​		***bloqué ne peut pas 	passer***
  - ​		***autorisé peut passer***
- ​	***Faites corriger par l'enseignant !***

Liens pour idées année prochaine :

https://www.cyberciti.biz/faq/how-to-use-iptables-with-multiple-source-destination-ips-addresses/

https://www.cyberciti.biz/faq/linux-traffic-shaping-ftp-server-port-21-22/

https://www.cyberciti.biz/faq/linux-howto-check-ip-blocked-against-iptables/

https://www.cyberciti.biz/faq/linux-configuring-ip-traffic-accounting/

https://www.cyberciti.biz/faq/linux-traffic-shaping-using-tc-to-control-http-traffic/

https://www.cyberciti.biz/faq/iptables-connection-limits-howto/

https://www.cyberciti.biz/faq/linux-port-redirection-with-iptables/

https://www.cyberciti.biz/faq/configure-iptables-to-allow-deny-access-to-samba/

https://www.cyberciti.biz/faq/ipv6-apache-configuration-tutorial/

https://www.cyberciti.biz/faq/linux-bastion-host/

https://www.cyberciti.biz/faq/how-to-save-restore-iptables-firewall-config-ubuntu/

https://www.cyberciti.biz/faq/linux-iptables-multiport-range/

https://www.cyberciti.biz/faq/block-entier-country-using-iptables/

https://www.cyberciti.biz/faq/ip6tables-ipv6-firewall-for-linux/

https://www.cyberciti.biz/faq/debian-iptables-stop/

https://www.cyberciti.biz/faq/howto-start-iptables-under-rhel-centos-linux/

https://www.cyberciti.biz/faq/display-iptables-router-nat-connections-using-netstat-nat/

https://www.cyberciti.biz/faq/linux-iptables-firewall-open-port-80/

https://www.cyberciti.biz/faq/ip_conntrack-table-ful-dropping-packet-error/

https://www.cyberciti.biz/faq/linux-demilitarized-zone-howto/

https://www.cyberciti.biz/faq/linux-open-iptables-firewall-port-22-23/

https://www.cyberciti.biz/faq/howto-block-ipaddress-of-spammers-with-firewall/

https://www.cyberciti.biz/faq/test-iptables-script-remotely/

[https://www.cyberciti.biz/faq/iptables-passive-](https://www.cyberciti.biz/faq/iptables-passive-ftp-is-not-working/)[**ftp**](https://www.cyberciti.biz/faq/iptables-passive-ftp-is-not-working/)[-is-not-working/](https://www.cyberciti.biz/faq/iptables-passive-ftp-is-not-working/)

https://www.cyberciti.biz/faq/howto-configure-network-address-translation-or-nat/

https://www.cyberciti.biz/faq/iptables-is-not-sending-log-to-syslog-file/

https://www.cyberciti.biz/faq/how-do-i-save-iptables-rules-or-settings/

https://www.cyberciti.biz/faq/iptables-open-ftp-port-21/

https://www.cyberciti.biz/faq/restrict-ssh-access-use-iptable/

https://www.cyberciti.biz/faq/tag/iptables/

#  Rappels

##   Le routage

Ci-dessous un rappel de commandes utilisée pour le routage IPv4.

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

##  Linux

### Les variables dans scripts

```bash
lan1if="eth0"
echo $lan1if
iptables -A INPUT -i $lo -s 127.0.0.1 -j ACCEPT
```

### Rappels sur le démarrage automatique d'un service Systemd

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

#  Préparation du laboratoire

##  Topologie à préparer

### Schéma

PC1 et PC2 sont les ordinateurs respectifs d’un binôme d’étudiants. Les topologies sur PC1 et PC2 sont identiques et sont réalisées à l’aide de VMs.

Dans chaque binôme, un étudiant réalise l’ensemble des exercices sur la topologie du PC1, tandis que l’autre que sur la topologie du PC2.

Les exercices sont individuels sauf lorsqu’il faut communiquer avec les réseaux LAN d’un voisin de gauche ou droite.

![image-20211020135755634](image-20211020135755634.png)

### Détails

#### Réseaux

X=1 pour les VM de PC1

X=2 pour les VMS de PC2

- Lan1 : Lan des « clients »
  - 172.16.X.0/24
  - Client live linux ou autre
  - ip dynamique dhcp du serveur
  - réseaux interne intnet1
- lan2 : Lan des « serveurs »
  - 192.168.X.0/24
  - Serveur Linux
  - ip statique 
  - réseaux interne intnet2
- Lan entre PC1 et PC2 
  - Adressage réseau du local

#### Client

- Iface
  - dhcp
  - intnet1
- clients
  - WWW
  - FTP
  - SSH
  - DHCP

#### Services

- VM linux fw
  - 3 ifaces
    - Type bridge  10.1.31.X(Mettre en statique l'IP reçue par le DHCP) vers le réseau extérieur
    - Réseau interne  		
      - Intnet1 pour le lan1 (172.16.X.254/24)
      - Intnet2 pour le lan2 (192.168.X.254/24)
  - services
    - proxy
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

##  Check list de vérification de la mise en place

1. ​	Réaliser une version papier de sa topologie avec les IP, interfaces, physiques et virtuelles, etc.
2. ​	Mettre en place le routage entre les 2 réseaux intnet.
3. ​	Les Fw doivent également pouvoir accéder à Internet.
4. ​	Les services FTP et DHCP doivent être fonctionnels et accessible.
5. ​	Le routage fonctionne avec les réseaux du voisin.

Ajout des routes statiques :

A corriger/vérifier avec le FW désactivé !

#  Règles de base d’un Firewall 

##  Syntaxe d'Iptable

### Anatomie - syntaxe générale

#### Aide

```bash
iptables -h| --help
man iptable
```

#### Syntaxe générale

```bash
iptables  [-t table] commande critère action
```

- ​	commande : Tâche à réaliser (ajouter une chaîne, etc.)
- ​	critère : critère de filtrage
- ​	action :

#### Visualiser

lister les règles :

```bash
iptables -L [-t table]
```

Compteurs de paquets[1](#sdfootnote1sym) :

Lister les règles:

```bash
iptables -L -v -t table [chaîne]
-x : valeur exacte, non arrondie
```

Remise à zéros des compteurs

```bash
-Z [chaîne]
```

### Règles

Une règle est une ligne ajoutant un comportement au firewall sur base de critères de filtrage

- Comportement : défini par une cible (target)
  - log
  - drop
  - etc.
- ​	Critères :
  - adresse IP
  - protocole
  - etc.

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
-R chaine n° règle (Cfr –line-numbers)
```

ex :

```bash
iptables –R 2 INPUT -p icmp --icmp-type echo-reply  -j ACCEPT 
```

#### Suppression

```bash
-D chaîne [n° règle | régle]
```

#### Insertion

```bash
-I chaîne [num] règle (pas num = 1 sous entendu)
```

#### Vider une table

```bash
iptables [-t table] -F [chaîne]
```

#### Remise à zéro des règles

```bash
iptables -Z chaine
```

#### Stratégie par défaut

```bash
iptables -t table -P chaîne action (action : ACCEPT; DROP)
ex :
iptables -P chain DROP
```

#### Cibles

```bash
iptables -A <…> -j Target
```

#### Critères

```bash
-p protocol
-s source address
-d destination address
-i input interface
-o output interface 
-f fragmented
```

#### Modules

```bash
iptables -m module -...
```

### Connection tracking

```bash
-m state --state : gérer l'état d'un connexion
NEW : 1er paquet de la connexion
ESTABLISHED : paquets suivants de la connexion
RELATED: paquets en relation avec la connexion
INVALID : paquets dont l'état n'as pas pu être déterminé
```



```bash
for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
iptables -A $j -m state --state ESTABLISHED,RELATED -j ACCEPT
done
```

### Exemples 

Les exemples ci-dessous prennent en compte un firewall statefull !

#### N'accepter que les requêtes de ping, les réponses étant traitées par le module "state"

```bash
for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
iptables -A $j -m state --state ESTABLISHED,RELATED -j ACCEPT
done
iptables -A INPUT -s $netw -p icmp --icmp-type echo-request -j ACCEPT
```

→ les réponses au ping seront autorisées grâce au tracking !

Attention ! : Utiliser une seule règle pour la gestion des sessions

Permet de simplifier les règles du FW

Mais peut mener à des failles dans la sécurité !

#### Accepter le trafic de la boucle locale[2](#sdfootnote2sym)

```bash
iptables -t filter -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```

#### Aide sur un protocole : 

```bash
iptables -p -icmp -h
```

#### Laisser les ping's traverser le firewall :

```bash
iptables -t filter –A FORWARD -p icmp --icmp-type echo-request  -j ACCEPT
```

#### Boucles via Bash : Stratégie par défaut et RAZ des chaînes

```bash
for i in nat mangle filter
do
iptables -t $i -F
done
for j in INPUT OUPUT FORWARD
do
iptables -P $j DROP
done
```

#### Autres

https://www.cyberciti.biz/tips/linux-iptables-examples.html

##  Manipulation : réaliser un script de firewall statefull avec emploi de variables

Configurez le firewall en respectant les demandes ci-dessous :

1. ​	Firewall statefull
2. ​	Lancé automatiquement au démarrage du système par systemd
3. ​	Emploi de variables pour les IP's, interfaces, etc.
4. ​	Remet à blanc iptables (R à z tables et compteurs)
5. ​	Accepte les requêtes DHCP pour le lan1
6. ​	Autorise ping depuis lan1 vers lan2
7. ​	Autorise ping vers le fw depuis le lan1
8. ​	Autorise le trafic local (depuis votre IP et la loopback) du fw
9. ​	Le script affiche les règles pour l’ensemble des tables avec les compteurs de paquets

À partir des règles demandées ci-dessus, est-il possible pour le PC client d’accéder au réseau externe ou à Internet ? Pourquoi ?

##  Manipulation : ajouter des règles supplémentaires au script précédent

Faites une copie de sauvegarde du script précédent et modifiez-le :

1. ​	Autorisez de pinger « le monde » depuis le routeur, mais pas le contraire
2. ​	Autorisez d’administrer les machines du lan2 depuis le lan1 (ssh)
3. ​	Autorisez au firewall d’être administré par la machine du lan2  
4. ​	autorisez l’accès au web (surfer) au lan1 et 2 en un minimum de lignes.  
5. ​	Autorisez l’impression sur 10.1.31.250 depuis le lan 1

L’accès à l’imprimante, ou à internet est-il possible ? Pourquoi?

##  Manipulation : règles supplémentaires concernant du trafic entrant et sortant de réseaux extérieurs et des services supplémentaires

Faites une copie de sauvegarde du script précédent et modifiez-le

Rem : Si vous autorisez du trafic vers/depuis le fw du voisin de gauche et/ou droite, Il faut  penser à ajouter les règles correspondantes sur l'autre firewall pour autoriser le trafic « en tant que voisin »

1. Ajoutez les routes statiques vers les lan 1 et 2 de votre voisin si ce n'est pas déjà fait. Désactivez le firewall et testez le routage.
2. Autorisez le trafic courriel en sachant que les services sont dans le lan2. Les clients sont dans le LAN1.
3. Votre firewall est administrable par le firewall de votre voisin [3](#sdfootnote3sym)
4. L’hôte de **votre** lan2 est joignable par le lan1 du **voisin** en ftp (installer un serveur et un client) **passif et actif**. N'oubliez pas d’utiliser la table raw et la chaîne PREROUTING avec la cible CT pour activer l’helper FTP du connection tracking (Cfr références).

Pourquoi une seule ligne pour ftp (avec le port 21) est-elle suffisante ?

Erreur typique : oublier d'installer et configurer le serveur ftp !

# Les chaînes personnalisées

##  Création d'une chaîne personnalisée

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

#### Conventions :

toujours en minuscules pour une chaîne personnalisée

en majuscule chaîne prédéfinie

##  Manipulation : création et utilisation d'un chaîne personnalisée

- ​	Ajouter une chaîne personnalisée qui filtre les paquets tcp avec tous les flags à 1 en même temps  sont bloqués (d'autres combinaisons de flags interdites existes, elles sont citées dans des listes) (Cfr explications module TCP)

#  Le NAT

##  Configuration du Nat

- ​	Il ne pas oublier de configurer la table filtrer également pour autoriser le trafic
- ​	Seul le 1er paquet est traité par le NAT. Les suivants le sont par le connection tracking
- ​	Il peut être nécessaire de charger un module pour le support de certains protocoles, comme le FTP.

### SNAT

#### Ip statique

```bash
iptables -t nat -A POSTROUTING -o eth0 -j SNAT –to-source 10.1.1.1

iptables -t nat -A POSTROUTING -p tcp -o eth0 -j SNAT --to-source 10.1.1.1-10.1.1.20:1024-32000
```

#### Ip dynamique

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### Le DNAT

#### DNAT

```bash
iptables  ...  --to-destination <ipaddr>[-<ipaddr>][:port-port] 
```

```bash
iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.1.1-192.168.1.10
```

#### REDIRECT

dnat vers un processus local : 

```bash
iptables  ...    --to-ports <port>[-<port>] 
```

##  Manipulation : Ajout du NAT

### SNAT

- Ajouter le NAT pour que les clients du lan1 puissent accéder au Web[5](#sdfootnote5sym).

### DNAT

- Manipulation : Ajout du DNAT sur le firewall pour que le service WEB du serveur soit accessible de l'extérieur


#  Proxy

##  Configuration de Squid

Rappel : faites une copie de sauvegarde du fichier de configuration au cas où …

port d'écoute par défaut :

```bash
3128
```

Configuration minimum :

```bash
/etc/squid.conf

acl our_networks src 10.1.6.0/24 192.168.2.0/24

http_access allow our_networks
http_access allow localhost

[http_access deny all]
```

proxy http transparent :

```bash
http_port … intercept
```

##  Manipulation : proxy obligatoire 

1. ​	Le proxy est situé sur le firewall

2. ​	Le pc client doit passer obligatoirement par un proxy pour accéder au WEB

   - Seul le proxy à la droit de 	sortir en http et https
   - Les hôtes n'ont accès en 	http et https qu'au proxy


##  Manipulation : proxy transparent

1. ​	le proxy est « transparent »
   - ​		Seuls les paquets http sont 	redirigés (NAT) vers le proxy !
   - ​		Les hôtes[6](#sdfootnote6sym) 	ne sont pas configurés pour utiliser un proxy
   

#  IP Accounting

##  Configuration avec iptables

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

##  Manipulation : Ip accounting simple

Remettez à 0 les règles du FW. Tout le trafic peut passer, mais il fait du Nat pour le lan1[7](#sdfootnote7sym) comme précédemment et fait de l'accounting :

1. Compter le trafic traversant 	le FW venant du LAN1
2. Compter séparément le 	trafic via une chaîne personnalisée qui regroupe :
   - DNS
   - Http
   - Https
   - Vers le portail henallux
3. Grâce à la commande watch, 	afficher en temps réel le trafic









[1](#sdfootnote1anc)Très utile pour déboguer en vérifiant par quelle règle les paquets passent.

Aussi pour optimiser les performances en mettant en 1er les règles qui traitent le plus de paquets

[2](#sdfootnote2anc) Il faut accepter le trafic qui sortant et rentrant comme il s'agit de 2 flux <> !

[3](#sdfootnote3anc) Ne pas oublier d'autoriser votre firewall a administrer votre voisin de droite !

[4](#sdfootnote4anc) Ne pas oublier d'autoriser votre firewall a administrer votre voisin de droite !

[5](#sdfootnote5anc) Ne pas oublier  DNS et https !

[6](#sdfootnote6anc) Il doivent donc avoir accès à un serveur DNS !

[7](#sdfootnote7anc) Récupérez les règles nécessaires dans les exercices précédents