---
layout: default
title: "Mise en place du laboratoire"
permalink: /mise-en-place/
---

# Mise en place du laboratoire

---
## Résumé

|                 |                                                           |
| --------------- | --------------------------------------------------------- |
| **But**         | Préparer le laboratoire                                   |
| **Objectifs**   | - Mettre en place une topologie réseau minimale. <br>     |
| **Parties**     | - Organisation <br>- Rappels <br>- Préparation <br>       |
| **Laboratoire** | - Mise en place de la topologie du laboratoire            |

---

## Section I — Organisation

### Objectifs de la section
Après cette section, vous serez capable de :

- Consulter les documentations officielles  
- Vous organiser pour le laboratoire

---

### Références

#### Documentation Officielle
- [Netfilter.org](https://www.netfilter.org/)
- [iptables extensions](http://ipset.netfilter.org/iptables-extensions.man.html)
- [iptables manual](http://ipset.netfilter.org/iptables.man.html)

#### Guides
- [Guide complet iptables (frozentux)](https://www.frozentux.net/documents/iptables-tutorial/) — la référence.
- [Connection tracking et usage sécurisé](https://home.regit.org/netfilter-en/secure-use-of-helpers/)
- [Iptables et vsftpd](https://wiki.archlinux.org/index.php/Very_Secure_FTP_Daemon)

#### Chez Debian
- [Wiki Debian - iptables](https://wiki.debian.org/iptables)
- [Debian Handbook - Firewall](https://debian-handbook.info/browse/stable/sect.firewall-packet-filtering.html)

#### Commandes utiles
- [DigitalOcean : iptables essentials](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands)

---

### Matériel
La manipulation peut être réalisée **entièrement en machines virtuelles** — un seul PC suffit.

> 💡 Il est vivement conseillé de **travailler en binôme**, surtout pour les manipulations impliquant deux firewalls.

La préparation des VMs est à **terminer à domicile avant la 2ᵉ séance**.

---

### Séances

Les explications des énoncés sont un résumé des références.  
N’hésitez pas à les consulter !

Pour chaque manipulation :

- **Commentez vos scripts !**  
- **Ne gardez aucune règle inutile.**  
- **Utilisez les filtres les plus précis possibles (IP, ports, interfaces, etc.).**  
- **Commentez le rôle de chaque ligne.**  
- **Vérifiez systématiquement :**  
  - que le trafic **bloqué** ne passe pas  
  - que le trafic **autorisé** passe bien  
- **Faites corriger par l'enseignant.**

---

## Section II — Rappels

### Objectifs de la section
Après cette section, vous serez capable sous Linux de :

- Configurer le routage  
- Utiliser des variables dans des scripts  
- Utiliser **Systemd** pour créer une unité avec démarrage automatique

---

### Configuration du routage

Rappel de commandes pour le routage IPv4 (principe identique pour IPv6) :

```bash
# cat /proc/sys/net/ipv4/ip_forward
1

# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

# cat /etc/sysctl.conf
...
net.ipv4.ip_forward = 0
...

# ip route show
# ip route add 192.168.55.0/24 via 192.168.1.254

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

---

### Développement d’un calcul IP

```bash
# man ipcalc
# ipcalc 192.168.0.1/24

Address:   192.168.0.1
Netmask:   255.255.255.0 = 24
Wildcard:  0.0.0.255
Network:   192.168.0.0/24
HostMin:   192.168.0.1
HostMax:   192.168.0.254
Broadcast: 192.168.0.255
Hosts/Net: 254   Class C, Private Internet
```

---

### Variables dans un script Bash

```bash
lan1if="eth0"
echo $lan1if
iptables -A INPUT -i $lan1if -s 127.0.0.1 -j ACCEPT
```

---

### Démarrage automatique d’un service via Systemd

La plupart des distributions utilisent **systemd** pour gérer le démarrage et les services.

L’idéal est de créer deux scripts :
- un pour **arrêter le firewall** (RAZ des règles)
- un pour **charger les règles**

Puis, créer la "unit file" `fw.service` dans `/etc/systemd/system/` :

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

Lancer, arrêter ou activer le service via :

```bash
systemctl enable fw.service
```

---

## Section III — Préparation de la topologie

### Objectif
Mettre en place une topologie de base avec routeurs, LANs, clients et serveurs.

---

###  Schéma du laboratoire

PC1 et PC2 sont les ordinateurs respectifs d’un binôme d’étudiants.  
Les topologies sur PC1 et PC2 sont identiques (réalisées via VMs).

> Chaque étudiant travaille sur son PC, mais les exercices de communication inter-LAN se font entre voisins.

![Schéma du labo]({{ '/assets/images/schema-labo.png' | absolute_url }})

---

### Réseaux

- **X=1** → PC1  
- **X=2** → PC2

**LAN1 :** Réseau “clients”  
- `172.16.X.0/24`  
- IP dynamique (DHCP)  
- Réseau interne `intnet1`

**LAN2 :** Réseau “serveurs”  
- `192.168.X.0/24`  
- IP statique  
- Réseau interne `intnet2`

**Lien PC1 ↔ PC2 :**  
- Adressage réseau du local

---

### Client

- Interfaces : DHCP et réseau `intnet1`

---

### Services

**VM Linux Firewall**  
- 3 interfaces :
  - Bridge → `10.1.31.X` (IP statique)
  - Intnet1 → LAN1 (`172.16.X.254/24`)
  - Intnet2 → LAN2 (`192.168.X.254/24`)
- Services :
  - Proxy
  - DHCP + routage
  - SSH

**Serveur**  
- Interface : Intnet2 (`192.168.X.1/24`)
- Services : WWW, FTP, SSH

---

### Mise en pratique

1. Installez et configurez les services sur le serveur et le routeur.  
2. Réalisez un schéma papier avec IPs, interfaces, etc.  
3. Activez le routage entre les réseaux client et serveur.  
4. Configurez le réseau pour que les firewalls aient accès à Internet.  
5. Créez les routes statiques (elles seront plus tard intégrées au script du firewall).

---

## Synthèse

Au cours de ce chapitre, vous avez appris à :

- Vous organiser pour ce cours  
- Mettre à profit des concepts (routage, variables, systemd)  
- Préparer un système Linux pour devenir un firewall  
- Configurer une topologie réseau minimale  
