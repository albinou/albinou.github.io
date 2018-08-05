---
layout: post
title:  "Accéder en SSH aux machines distantes d'un LAN"
author: Albin Kauffmann
---

C'est malheureux, mais l'IPv4 a encore de beaux jours devant lui.
Si vous avez la chance d'avoir une connexion IPv6 à la maison, vous trouverez
peu d'hôtels et de points d'accès Wi-Fi publics vous offrant une adresse IPv6.
Alors que faire pour accéder facilement aux machines de votre LAN depuis
l'extérieur ?

Le problème est abordé ici avec la topologie réseau illustrée ci-dessous.

```
                                        ,--------------,
                                      ,-| machine1.lan |
                                      | `--------------'
,------------,   ,------------------, | ,--------------,
|  INTERNET  |---| home.exemple.org |-|-| machine2.lan |
`------------'   `------------------' | `--------------'
                                      | ,--------------,
                                      '-| machine3.lan |
                                        `--------------'
```
Notez que :

* _home.exemple.org_ est l'entrée DNS de la gateway recevant les connexions IPv4
entrantes (cette gateway reçoit les connexions entrantes du port SSH)
* _machineX.lan_ sont les entrées DNS locales correspondant à des adresses
IPv4 privées (rien n'empêche cependant d'utiliser des addresses IPv6 locales)

Je vous propose ici 2 solutions.
Si vous avez une autre solution, n'hésitez pas à la partager dans les
commentaires.

## 1<sup>ère</sup> solution : Configurer un proxy avec son client SSH

Le client SSH permet de configurer une commande «proxy» (option _ProxyCommand_).
Ceci permet de créer un tunnel SSH dans lequel sera transmis le flux SSH du
serveur final à joindre (_machineX.lan_).

Chaque client peut donc être configuré avec les 2 lignes suivantes dans le
fichier _~/.ssh/config_ du client SSH :

```
Host machine1.lan machine2.lan machine3.lan
	ProxyCommand ssh home.exemple.org -W %h:%p
```

### Avantages

* Simple à mettre en place

### Inconvénients

* Nécessite un compte UNIX sur la gateway _home.exemple.org_
* Nécessite de taper 2 fois le mot de passe si vous n'utilisez pas d'authentification par clef
* L'adresse IP de connexion visible dans le journal de _machineX.lan_ sera celle de la gateway (_home.exemple.org_)

## 2<sup>ème</sup> solution : Configurer le NAT du pare-feu (routeur, _iptables_ ou _nftables_)

Cette deuxième technique demande un peu plus de connaissance puisqu'il faut
jouer avec les règles de pare-feu.
L'objectif est de créer une règle DNAT («Destination NAT») modifiant la
destination des paquets entrants sur un certains ports (à choisir librement
mais supérieurs à 1024 idéalement).

Pour cela, 3 possibilités :

* si le traffic entrant traverse un routeur commercial (ou la «box» fournie par
un opérateur), il est sans doute possible de configurer une redirection de port
dans son interface Web
* si la gateway utilise _iptables_ pour d'autres tâches, il est possible de
configurer un DNAT avec _iptables_ (de nombreux tutoriaux sont disponibles sur
Internet)
* si la gateway utilise _nftables_, je vous propose ci-dessous une configuration
_nftables_ définissant un DNAT sur une liste de ports à rediriger :

```
table ip nat {
	map ssh_ports {
		type inet_service : ipv4_addr
		elements = { \
			8251 : 192.168.1.11, \
			8252 : 192.168.1.12, \
			8253 : 192.168.1.13  \
		}
	}

	chain prerouting {
		type nat hook prerouting priority 0;
		iif eth-wan dnat to tcp dport map @ssh_ports:ssh
	}
}
```

Cette configuration permet de rediriger les connexions entrantes sur les ports
8251, 8252 et 8253 respectivement vers les adresses 192.168.1.11, 192.168.1.12
et 192.168.1.13 (correspondantes aux _machineX.lan_).
Explications de ce code :

1. On définit une structure de données _ssh_ports_ contenant une correspondance
entre ports et adresses de destination
2. On définit qu'avant de router chaque paquet entrant (phase de _pre-routage_ /
_prerouting_) sur le port _ssh_, on change son port de destination si celui-ci
se trouve dans _ssh_ports_

Ainsi, pour se connecter à _machine2.lan_, il faudra utiliser la commande
suivante :

```shell
ssh -p 8251 home.exemple.org
```

Il est également possible de définir la configuration suivante dans son
_~/.ssh/config_ :

```
Host machine1.lan
	Hostname home.exemple.org
	Port 8251

Host machine2.lan
	Hostname home.exemple.org
	Port 8252

Host machine3.lan
	Hostname home.exemple.org
	Port 8253
```

Pour ainsi n'avoir que la commande suivante à taper :

```shell
ssh machine1.lan
```

### Avantages

* Ne nécessite pas de compte sur la gateway _home.exemple.org_
* Offre sans doute un meilleur débit pour les transferts de fichier en SFTP car pas d'encapsulation

### Inconvénients

* Rend le serveur SSH des machines du LAN directement accessible depuis Internet (attention à la sécurité)

## Ressources

* Manuel de ssh : <http://man7.org/linux/man-pages/man1/ssh.1.html>
* Manuel de ssh_config : <http://man7.org/linux/man-pages/man5/ssh_config.5.html>
* Page Wikipédia sur le NAT : <https://fr.wikipedia.org/wiki/Network_address_translation>
* Manuel de iptables : <http://ipset.netfilter.org/iptables-extensions.man.html>
* Wiki de nftables : <https://wiki.nftables.org>
