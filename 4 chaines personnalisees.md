---
layout: default
title: "Caneva de Script"
permalink: /caneva-script/
---

# Script de configuration iptables

Ce document contient le script shell fourni, mis en forme pour une lecture plus confortable.  
Le code ci-dessous définit des règles de base pour iptables en réinitialisant les chaînes et en appliquant une politique restrictive par défaut.

```bash
#!/bin/bash
#FLUSH ALL
iptables -F INPUT
iptables -F OUTPUT
iptables -F FORWARD
iptables -t nat -F PREROUTING
iptables -t nat -F POSTROUTING 

###################### #filter# ###################### 
### INPUT RULES ### 
#Règles INPUT 

#Bloquer le reste 
iptables -P INPUT DROP
### FORWARD RULES ### 
#Règles FORWARD 

#Bloquer le reste
iptables -P FORWARD DROP
### OUTPUT RULES ###


iptables -P OUTPUT DROP

###################### #nat# ###################### 
###Dnat###
#Règles de dnat 
###Snat###
#Règles de snat
```
