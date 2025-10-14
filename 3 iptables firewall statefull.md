# Firewall statefull

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

* mettre en place un firewall statefull

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

## Section II : Modules et helpers

### Objectifs de la section

Pour des protocoles particuliers:

* Utiliser les modules
* Configurer les règles
* Activer les helpers

### Les modules

L'emploi de modules explicites est obligatoires pour certains protocoles, afin que le firewall puisse les interpréter et exploiter le connection tracking afin de les gérer.

Dans certains cas, il faudra d'abord charger le module kernel (pour le filtrage ou le NAT ) avec `modprobe`.

Si la bersion du kernel est 2.6.19 ou antérieure, les modules se nomment `ip_conntrack_...` ou `ip_nat...`. Sinon ce sera `nf_conntrack...` ou `nf_nat_...`

```bash
uname -r
5.15.0-52-generic

lsmod | grep sip

modprobe nf_conntrack_sip nf_nat_sip

lsmod | grep sip
...
nf_nat_sip
nf_conntrack_sip
...
```

Une fois ces modules chargé, une configuration correcte du connexion tracking permettra le support de ces protocoles (sessions "`related`") pour le filtrage ou le NAT.

### Les helpers

Depuis le kernel 4.7, les helpers ne sont plus activés par défaut sur les sessions. Il faut désormais passer des règles qui utilisent la cible `CT` dans la chaîne `PREROUTING` de la table `raw` .

```bash
iptables -A PREROUTING -t raw …… -j CT --helper sip
```

## Laboratoire :  réaliser un script de firewall statefull

### Firewall local

A partir du laboratoire précédent, créez un script de firewall qui respecte les demandes ci-dessous.

1. Règles de base pour obtenir un firewall statefull.
2. Démarrage par systemd. Lors de l'arrêt par systemd, les règles sont bien remise à zéro.
3. Variables pour les IP's, interfaces, protocoles, etc.
4. Remise à blanc iptables (R à z table**s**, compteurs, etc.).
5. Autoriser le trafic local (depuis votre IP et la loopback) du fw.
6. Accepter les requêtes DHCP pour le firewall.
7. Autoriser de pinger « le monde » depuis le routeur, mais pas le contraire.
8. Autoriser ping vers le fw depuis le lan1.
9. Afficher les règles pour l'ensemble des tables avec les compteurs de paquets.

### Firewall réseau

Faites une copie de sauvegarde du script précédent et modifiez-le en ajoutant :

1. Le routage est pris en charge par le script et non plus par la configuration du réseau. 
2. Lors de l'arrêt par systemd, les routes sont supprimées.
3. Autoriser ping depuis lan1 vers lan2.
4. Autoriser l'administration des machines du lan2 depuis le lan1 (ssh).
5. Autoriser au firewall d'être administréLes par la machine du lan2  .
6. Autoriser l'accès au web (surfer) au lan1 et 2 en un minimum de lignes.
7. Autoriser l'impression sur 10.1.31.250 depuis le lan 1. Simulez en choisissant un protocole d'impression au choix.

À partir des règles demandées ci-dessus, est-il possible pour le PC client d'accéder au réseau externe, à Internet ou à l'imprimante? Pourquoi ?

### Firewall avec l'emploi de helpers et réseau avec plusieurs firewalls

Modifiez les scripts précédents après avoir réalisé une copie:

1.  Ajoutez les routes statiques vers les lan 1 et 2 de votre voisin si ce n'est pas déjà fait.

2. Autorisez le trafic courriel

   1. les services sont dans le lan2
   2. Les clients sont dans le LAN1
   3. protocoles : SMTP, IMAP, POP

   > N'oubliez pas que vos règles doivent permettre d'envoyer et recevoir des courriels du "monde extérieur"

3. ​	Votre firewall est administrable par le firewall de votre voisin [1](#sdfootnote1sym)

4. ​	L'hôte de **votre** lan2 est joignable par le lan1 du **voisin** en ftp **passif et actif**. 

Pourquoi une seule ligne pour ftp (avec le port 21) est-elle suffisante ?

## Synthèse

Au cours de ce chapitre, vous avez appris : 

* gérer les règles d'un firewall Iptables;
* écrire un script de firewall Iptables statefull qui sera démarré automatiquement;
* créer les règles d'un Firewall statefull qui utilise des protocoles particuliers et des helpers.

[1](#sdfootnote1anc) Ne pas oublier d'autoriser votre firewall a administrer votre voisin de droite !

