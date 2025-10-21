#  Préparation du laboratoire

##  Check list de vérification de la mise en place

1. ​	Réaliser une version papier de sa topologie avec les IP, interfaces, physiques et virtuelles, etc.
2. ​	Mettre en place le routage entre les 2 réseaux intnet.
3. ​	Les Fw doivent également pouvoir accéder à Internet.
4. ​	Les services FTP et DHCP doivent être fonctionnels et accessible.
5. ​	Le routage fonctionne avec les réseaux du voisin.

```
# Ajout des routes statiques :
# A corriger/vérifier avec le FW désactivé !

# dans le fichiers "/etc/interfaces", ajouter  en fin de section de la carte réseau de sortie des routes :

up ip route add x.x.x.x/24 via  x.x.x.x
down ip route del x.x.x.x/24 via  x.x.x.x
```



Laboratoire :  réaliser un script de firewall minimal avec emploi de variables

# Réaliser un script de firewall minimal avec emploi de variables

## Firewall local avec les règle de bases

Script `stop.sh`

```bash
#!/bin/bash

echo "*******************************************"
echo "cleanning rules et resetting default policy"
echo "*******************************************"

for j in INPUT OUTPUT FORWARD
do
iptables -P $j ACCEPT
done
for i in nat raw filter
do
iptables -t $i -F
done
```

Script `start.sh`

```bash
#!/bin/bash

lo="lo"
lan1if="eth1"
lan2if="eth2"
wanif="eth0"
...

source stop.sh

for j in INPUT OUTPUT FORWARD
do
iptables -P $j DROP
done

for i in filter nat raw
do
echo "**********************"
echo "      Table $i "
echo "**********************"

iptables -L -v -n --line-numbers -t $i
done 


```

Ensuite, il faut créer la "unit file", par exemple `fw.service` dans le répertoire `/etc/systemd/system/`

```bash
[Unit]
Description=Add Firewall Rules to iptables

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/etc/firewall/enable.sh
ExecStop=/etc/firewall/stop.sh

[Install]
WantedBy=multi-user.target 
```

Rendre les scripts exécutables

```bash
chmod 770 ...
```

Lancer, arrêter, etc, le service se fait via la commande habituelle, `systemctl `: 

```bash
systemctl daemon-reload
systemctl start fw.service
systemctl enable fw.service
```

#  Réaliser un script de firewall statefull 

## Firewall local

```bash
#!/bin/bash

#Définition des variables

lo="lo"
lan1if="eth1"
lan2if="eth2"
wanif="eth0"
lan1ip="172.16.0.0/24"
lan2ip="192.168.0.0/24"
wanip="10.1.6.0/24"
#client1 en DHCP → commenté
#client1ip="172.16.0.1"
client2ip="192.168.0.1"
ftpip="10.1.6.253"
printerip="10.1.6.250"
printerport="9100"
fwvoisinip= IP DU FW à administrer dans le réseau voisin


#Ajout du connection tracking

#Le mise en place du connection tracking doit toujours etre la 1ere règle car il va filtrer la plus grande partie de taffic !

#En effet, seul le 1er paquet est traité par les règles ci-après, les suivant de la session passant le connection tracking. Il faut donc mettre en 1er le trafic qui rencontre le + de nouvelles sessions en 1er ç à d le connexion tracking !!!!!!

for j in INPUT OUTPUT FORWARD
do
        iptables -P $j DROP	# stratégie par défaut à DROP
        iptables -A $j -m state --state RELATED,ESTABLISHED -j ACCEPT # Accepter les paquets des sessions existantes ou reliées
done

#Autorise le traffic local  du fw

iptables -A INPUT -i $lo -s 127.0.0.1 -j ACCEPT
iptables -A OUTPUT -o $lo -d 127.0.0.1 -j ACCEPT

# Accepte les requêtes dhcp pour le lan1. En pratique, comme les requêtes DHCP sont interceptée au niveau de la couche 2, iptables de les bloquent de tout façon pas

iptables -A INPUT -s $lan1ip -i $lan1if -m udp -p udp --dport 67 --sport 68 -j ACCEPT 

# Autorise ping vers le fw depuis le lan1

iptables -A INPUT -s $lan1ip -i $lan1if -p icmp --icmp-type echo-request -j ACCEPT 

#Permet de pinger « le monde » depuis le routeur, mais pas le contraire

iptables -A OUTPUT -o $wanif -m icmp -p icmp --icmp-type echo-request -j ACCEPT


```



##  Firewall réseau

Supprimer le routage de systctl

`start.sh`

```bash
#!/bin/bash

#couper le routage

echo 0 > /proc/sys/net/ipv4/ip_forward

##############################
# règles ex. précédents ne sont pas reprises dans le correctif
###############################
...

# accepter les ping lan 1 → lan2 

iptables -A FORWARD -s $lan1ip -i $lan1if -d $lan2ip -o $lan2if -m icmp -p icmp --icmp-type echo-request -j ACCEPT

#Permet d'administrer les machines du lan2 depuis le lan1 (ssh)

iptables -A FORWARD -s $lan1ip -d $lan2ip -i $lan1if -o $lan2if -p tcp -m tcp --dport 22 -j ACCEPT

#être administré par hote lan2

iptables -A INPUT -s $client2ip -d $lddfdf -i $lan2if -p tcp -m tcp --dport 22 -j ACCEPT

# Surfer -> traffic le plus rencontré, avec https en 1er, puis http et enfin dns
# Un peu moins d'importance ici (connexion tracking), mais toujours mieux de trier par ordre décroissant du nombre de paquet qui seront traités, surtout sur un FW qui traite bcp de traffic. D'où l'importance d'afficher le nombre de paquets traité par les règles

iptables -A FORWARD -i $lan1if -o $wanif -s $lan1ip -p tcp -m tcp --dport $https -j ACCEPT
iptables -A FORWARD -i $lan2if -o $wanif -s $lan2ip -p tcp -m tcp --dport $https -j ACCEPT
iptables -A FORWARD -i $lan1if-o $wanif -s $lan1ip -p tcp -m tcp --dport $http -j ACCEPT
iptables -A FORWARD -i $lan2if -o $wanif -s $lan2ip -p tcp -m tcp –dport $http -j ACCEPT

#les requêtes DNS sont uniquement en udp. TCP = pour synchro des serveurs

iptables -A FORWARD -i $lan1if -o $wanif -s $lan1ip -p udp -m udp --dport $dns -j ACCEPT
iptables -A FORWARD -i $lan2if  -o $wanif -s $lan2ip -p udp -m udp --dport $dns -j ACCEPT

#impression

iptables -A FORWARD -s $lan1ip -d $printerip -i $lan1if -o $wanif -p tcp --dport $printerport -j ACCEPT



# ajouter les routes

ip route add ...

# Activer le routage

echo 1 > /proc/sys/net/ipv4/ip_forward

# Il ne peut pas accéder à internet ou au réseau externe car le réseau du PC n'est pas connu de l'extérieur et il n'y donc pas de route de retour. Pour régler cela, il faudrait soit ajouter les routes de retour sur les routeurs "externes" à notre topologie, soit mettre en place du NAT


```

stop.sh

```bash
# ajouter 

ip route del ...
```



##  Firewall avec l'emploi de helpers et réseau avec plusieurs firewalls



```bash
# Ci-joint les <> par rapport à ex précédent :  dans le script FW

#Autoriser le trafic courriel en sachant que le service est dans le lan2. Traffic mail le 2ém en terme de BP

##SMTP sortant clients
iptables -A FORWARD -s $lan1ip -i $lan1if -o $lan2if -d $lan2ip -p tcp -m tcp --dport 25 -j ACCEPT

##SMTP pour le MX (courriels entrants et sortants entre le serveur et le "monde")
iptables -A FORWARD -s $lan2ip -i $lan2if -o $wanif -p tcp -m tcp --dport 25 -j ACCEPT
iptables -A FORWARD -d $lan2ip -o $lan2if -i $wanif -p tcp -m tcp --dport 25 -j ACCEPT

##POP3 
iptables -A FORWARD -s $lan1ip -i $lan1if -o $lan2if -d $lan2ip -p tcp --dport 110 -j ACCEPT

##IMAP 
iptables -A FORWARD -s $lan1ip -i $lan1if -o $lan2if -d $lan2ip -p tcp --dport 143 -j ACCEPT

#SSH : Votre firewall est administrable par le firewall de votre voisin 1
iptables -A INPUT -s $fwgaucheip -i $wanif -m tcp -p tcp --dport  22 -j ACCEPT

# ne pas oublier la regle qui permet la réciproque → administrer son voisin !
iptables -A OUTPUT -d $fwdroiteip -o $wanif -m tcp -p tcp --dport  22 -j ACCEPT

# L’hôte de votre lan2 est joignable par le lan1 du voisin en ftp (installer un serveur et un client) passif et actif. 

# modprobe nf_conntrack_ftp  nouveau nom, ip conntrack est désormais un alias (donc ça marche encore)
modprobe ip_conntrack_ftp 

# En cas de nat, il faudrait aussi activer le module ip_nat_ftp
#modprobe nf_nat_ftp

# Depuis le kernel 4.7, les helpers sont par défaut désactivés pour le connection tracking. Il faut dans la table raw ajouter une règles pour l’activer pour le protocole désiré
iptables -A PREROUTING -t raw -p tcp --dport 21 -d $....  -j CT --helper ftp

# Règle FTP
iptables -A FORWARD -s $lan1gaucheip -d $lan2ip -o $lan1if -i $wanif -p tcp -m tcp --dport 21 -j ACCEPT

# règle réciproque pour le FTP
iptables -A FORWARD -d $client2droiteip -s $lan1ip -i $lan1if -o $wanif -p tcp -m tcp --dport 21 -j ACCEPT


# Une seule ligne est nécessaire car le traffic // est géré par "RELATED"
```

#  Les chaînes personnalisées

##  Laboratoire : création et utilisation d'un chaîne personnalisée

```bash
# Ajouter une chaine personnalisée qui filtre les paquets tcp avec tous les flags à 1 en même temps  sont bloqués (d'autres combinaisons de flags interdites existes, elles sont citées dans des listes) (Cfr explications module TCP)

# dropper les les segments avec les flags TCP à 1 : ALL (masque pour indiquer la liste des flag à tester) ALL (liste des flag qui doivent être à 1 pour correspondre à la règle) → ALL ALL tout tester et tout doit être à un

iptables -N tcpfilter
iptables -A tcpfilter -m tcp -p tcp --tcp-flags ALL ALL -j DROP

#il reste à modifier la boucle avec les règles communes aux 3 chaînes de filter :

for j in INPUT OUTPUT FORWARD
do
        iptables -P $j DROP
        iptables -A $j -m state --state ESTABLISHED,RELATED -j ACCEPT        
#appel de la chaine tcpfilter
        iptables -A $j  -j tcpfilter
done
```

#  Le NAT

##  Laboratoire

```bash
# SNAT

## Activer le NAT en masquerade si ip dyn, sinon utiliser SNAT ! Attention cela ne fait que modifier l'IP de destination. Cela n'autorise rien ! Ne jamais mélanger du SNAT et du MASQUERADE

## Attention, le trafic vers le voisin ne marche plus! -> cf plus loin pour corriger

iptables -t nat -A POSTROUTING -i $lan1if -s $lan1ip -o $wanif  -j SNAT --to-source 10.1.31.X

# Autorisations

## HTTP
iptables -t filter -A FORWARD -i $lan1if -s $lan2ip  -o $wanif -m tcp -p tcp --dport 80 -j ACCEPT

## Proxy (si d'application)
iptables -t filter -A FORWARD -i $lan1if -s $lan2ip  -o $wanif -m tcp -p tcp --dport 8080 -j ACCEPT

## HTTPS
iptables -t filter -A FORWARD -i $lan1if -s $lan2ip  -o $wanif -m tcp -p tcp --dport 443 -j ACCEPT

## DNS
iptables -t filter -A FORWARD -i $lan1if -s $lan2ip  -o $wanif -m tcp -p tcp --dport 53 -j ACCEPT
iptables -t filter -A FORWARD -i $lan1if -s $lan2ip  -o $wanif -m udp -p udp --dport 53 -j ACCEPT

# Pour que le trafic soit routé vers le voisin, il ne faut pas nater pour le trafic à destination de ce réseau en modifiant la ligne du SNAT:

## iptables -t nat -A POSTROUTING -i $lan1if -s $lan1ip -o $wanif !-d $lanvoisin  -j SNAT --to-source 10.1.31.X

#DNAT vers le serveur Web

iptables -t nat -A PREROUTING -p tcp -i $wanif -d $wanip --dport $porthttp -j DNAT --to $httpserver

# Autoriser le traffic du DNAT

iptables -A FORWARD -p tcp -i $wanif -d $wanip --dport $porthttp -j ACCEPT
```

#  Proxy

##  Manipulation : proxy obligatoire 

```bash
# Iptables

## Autoriser le trafic en INPUT vers SQUID

iptables -t filter -A INPUT -i $lan1if -s $lan2ip -m tcp -p tcp --dport 3128 -j ACCEPT

## seul squid peut sortir en http (supprimer les règles voules et ajouter

iptables -A OUTPUT -p tcp -s $IP_PROXY  -o $WanIface –dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp -s $IP_PROXY  -o $WanIface –dport 443 -j ACCEPT
iptables -A OUTPUT -p udp -s $IP_PROXY  -o $WanIface –dport 53 -j ACCEPT
```

```bash
# Squid

## Proxy transparent

http_port … intercept

## Modifier et décommenter si nécessaire l'ACL qui contient la liste des SR :

acl our_networks src 10.1.6.0/24 192.168.2.0/24

## Autoriser les SR connus :
http_access allow our_networks
http_access allow localhost

## Interdire les autres réseaux :
    
http_access deny all
```

##  Manipulation : proxy transparent

```bash
# redirection des paquets http vers le proxy . Pour envoyer un paquet http, les clients doivent pouvoir faire du DNS vers l'extérieur ou passer via un cache DNS qui lui a accès à internet)

# Ajouter connection tracking, etc . comme précédemment

iptables -A PREROUTING -t nat -p tcp --dport 443 -i $LAN -s $LanIface -j REDIRECT --to-port 3128 # si le proxy est sur le FW, utiliser la cible REDIRECT, sinon DNAT
iptables -A PREROUTING -t nat -p tcp --dport 80 -i $LAN -s $LanIface -j REDIRECT --to-port 3128 # si le proxy est sur le FW, utiliser la cible REDIRECT, sinon DNAT


# seul squid peut sortir en http (supprimer les règles voules et ajouter )

iptables -A OUTPUT -p tcp -s $IP_PROXY  -o $WanIface –dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp -s $IP_PROXY  -o $WanIface –dport 443 -j ACCEPT
iptables -A OUTPUT -p udp -s $IP_PROXY  -o $WanIface –dport 53 -j ACCEPT
```

#  IP Accounting

##  Manipulation : Ip accounting simple

```bash
#mettre à 0 le fw

echo 0 > /proc/sys/net/ipv4/ip_forward

for i in raw nat mangle filter
do
        iptables -t $i -F
        iptables -t $i -Z
done

#Stratégie par défaut

for j in INPUT OUTPUT FORWARD
do
        iptables -P $j ACCEPT
done

#comptage nbre pkt venant du lan1 

iptable -t filter -A FORWARD -j FROMLAN1

# chaîne qui compte le trafic dns et web

iptables -N FROMLAN1
iptables -A  FROMLAN1 -s $lan1IP -i $lan1if -m udp -p udp --dport 53
iptables -A  FROMLAN1 -s $lan1IP -i $lan1if -m tcp -p tcp --dport 80
iptables -A  FROMLAN1 -s $lan1IP -i $lan1if -m tcp -p tcp --dport 443
iptables -A  FROMLAN1 -s $lan1IP -i $lan1if -m tcp -d $PortalHenalluxIP

echo 1 > /proc/sys/net/ipv4/ip_forward
```

```bash

# surveiller en continu : utiliser watch depuis la cmd
watch iptables -v -n -x -L FROMLAN1

```
