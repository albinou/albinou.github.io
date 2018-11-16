---
layout: post
title:  "Exécuter un programme dans un sandbox (firejail)"
author: Albin Kauffmann
---

Cet article aborde brièvement comment exécuter une application dans un bac à
sable (ou _sandbox_ en anglais).
L'intérêt ici est d'exécuter un programme en limitant les impacts :

* d'une attaque sur un service ou un logiciel (piratage)
* de l'exécution d'un logiciel de source non fiable (ex. logiciel propriétaire)

Pour cela cet article propose d'utiliser _firejail_, un outil qui permet
essentiellement de :

* limiter l'accès aux fichiers du système (_$HOME_ mais aussi des fichiers
spéciaux de _/dev_, _/proc_ et _/sys_)
* restreindre les appels systèmes et les _capabilities_ de Linux
* restreindre les protocoles réseaux

La configuration et l'utilisation de _firejail_ est cependant très facile car
cet outil est fourni avec des profils prédéfinis.

## Installation de _firejail_

Un paquet existe pour la plupart des distributions Linux.
Dans mon cas, sous _ArchLinux_ :

```bash
albinou@pc:~$ pacman -S firejail
```

Il est désormais possible de lancer n'importe quel programme avec la
configuration par défaut (définie dans _/etc/firejail/default.profile_) :

```bash
albinou@pc:~$ firejail echo 'Hello world!'
Reading profile /etc/firejail/default.profile
[...]
Hello world!

Parent is shutting down, bye...
```

## Profils _firejail_

La configuration par défaut n'est pas très limitante.
Elle n'a notamment aucune restriction sur l'espace disque de _$HOME_ qui peut
contenir des données sensibles.

Heureusement pour nous, _firejail_ fournit de nombreux profils par défaut.
Ils se trouvent dans _/etc/firejail/_ et il en existe un notamment pour le
navigateur _firefox_.
Il suffit de lancer `firejail firefox` pour que cette configuration soit
utilisée.
Cette configuration permet notamment de ne donner accès qu'au répertoire
_~/Downloads_ ainsi qu'au répertoire _~/.mozilla_ contenant la configuration de
_firefox_.

Pour valider que l'instance de _firefox_ est corrctement lancée dans une
sandbox, il est possible de lancer la commande suivante :

```bash
albinou@pc:~$ firejail --list
6423:albinou::/usr/bin/firejail /usr/bin/firefox
```

## Utilisation de _firejail_ par défaut

Pour que _firejail_ soit utilisé automatiquement (sans avoir besoin de
l'indiquer sur la ligne de commande), il suffit de créer un lien symbolique vers
_/usr/bin/firejail_ dans _/usr/local/bin/_ avec le nom du programme (qui
doit être le même que le nom du profil) :

```bash
albinou@pc:~$ sudo ln -s /usr/bin/firejail /usr/local/bin/firefox
```

**Attention** alors à bien positionner la variable _$PATH_ pour que les
programmes de _/usr/local/bin/_ soient prioritaires sur ceux de _/usr/bin/_.

Pour créer automatiquement un lien symbolique pour chaque application dont la
configuration existe dans _/etc/firejail/_, il suffit d'appeler _firecfg_ :

```bash
albinou@pc:~$ sudo firecfg
```

Cette commande va donc créer les liens symboliques dans _/usr/local/bin/_ mais
va également créer un fichier _/etc/firejail/firejail.users_ listant les
utilisateurs pour lesquels _firejail_ va être utilisé.
Pour utiliser _firejail_ pour tous les utilisateurs, le plus simple est de
supprimer ce fichier (cf. [_man 5 firejail-users_](http://man7.org/linux/man-pages/man5/firejail-users.5.html)
pour plus d'info) :

```bash
albinou@pc:~$ sudo rm /etc/firejail/firejail.users
```

## Ressources

* Site Web de firejail : <https://firejail.wordpress.com>
* Guide pour firefox : <https://firejail.wordpress.com/documentation-2/firefox-guide>
* Page Wiki de ArchLinux: <https://wiki.archlinux.org/index.php/Firejail>
* Manuel utilisateur de _firejail_ : <http://man7.org/linux/man-pages/man1/firejail.1.html>
* Manuel utilisateur de _firecfg_ : <http://man7.org/linux/man-pages/man1/firecfg.1.html>
* Manuel utilisateur de _firejail-users_ : <http://man7.org/linux/man-pages/man5/firejail-users.5.html>
