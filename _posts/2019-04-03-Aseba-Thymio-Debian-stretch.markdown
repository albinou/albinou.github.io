---
layout: post
title:  "Aseba pour Debian stretch (robot Thymio)"
author: Albin Kauffmann
---

<div markdown="1">

* TOC
{:toc}

![Robot Thymio]({{ "/assets/images/2019-04-03-thymio.jpg" | relative_url }})

</div>{: .float-right }

J'ai offert à mon neveu un robot [Thymio](https://www.thymio.org/fr:thymio).
Si vous ne connaissez pas, allez faire un tour sur la page des [spécifications](https://www.thymio.org/fr:thymiospecifications).

Afin de le programmer, il est nécessaire d'installer la suite logicielle [Aseba](https://www.thymio.org/fr:start) qui inclue notamment :

* Thymio VPL (Langage de Programmation Visuel)
* Blockly (programmation en blocs semblable à Scratch)
* Aseba Studio (programmation textuelle)

Mais malheureusement, cette suite n'est pas disponible dans les dépôts officiels de Debian stable (Debian *stretch* au jours où j'écris cet article).
À noter cependant, Aseba est disponible dans Debian testing (nom de code *buster*) et qu'elle sera donc disponible dans la future Debian stable (c.f. <https://packages.debian.org/buster/aseba>).

Dans cet article, je propose donc les étapes pour :

* installer le paquet Debian précompilés par mes soins
* la méthode que j'ai suivie pour créer ces paquets

## Installation sous Debian stretch

L'installation sous Debian stretch (ou toute distribution basée sur Debian stretch) nécessite 4 étapes détaillées ci-dessous :

1. Me faire confiance et ajouter mon certificat
1. Ajouter le dépôt *packages.kauff.org* contenant Aseba
1. Mettre à jour les dépôts de paquets
1. Installer Aseba

Pour cela, les commandes à exécuter sont les suivantes :

```bash
albinou@pc:~$ curl -s https://albin.kauff.org/albinkauffmann-pubkey.txt | sudo apt-key add -
OK
albinou@pc:~$ sudo bash -c 'echo "deb http://packages.kauff.org/debian/ stretch aseba" > /etc/apt/sources.list.d/packages-kauff-org.list'
albinou@pc:~$ sudo apt-get update
albinou@pc:~$ sudo apt-get install aseba
```

Pour tenter la version expérimentale (branche *master* de git), il est possible de modifier le fichier *packages-kauff-org.list* afin d'utiliser le composant aseba-git :

```
deb http://packages.kauff.org/debian/ stretch aseba-git
```

Le processus n'étant pas automatique pour moi, je ne promets cependant pas de régulièrement rebuilder la branche *master*.

## Construction des paquets (HowTo)

Les explications qui suivent sont très concises.
Pour plus de détails, ne pas hésiter à regarder les liens disponibles dans les [Ressources](#ressources) de cet article.

### Génération d'une clef GPG

Pour cette étape, je conseille un article de Nextinpact très bien écrit et disponible [ici](https://www.nextinpact.com/news/102685-gpg-comment-creer-paire-clefs-presque-parfaite.htm).

À la fin de l'étape de création des clefs, la commande `gpg --list-key` doit retourner une clef valide.

### Construction (gbp buildpackage)

Les paquets sont construits avec les commandes suivantes :

```bash
cd /PATH/TO/SOURCES
# Create a new branch for the build
git checkout -b build
# Update the Debian changelog to add a version (required if building master)
gbp dch --debian-branch=build --since=1.6.0 --release
# Build the package
gbp buildpackage --git-debian-branch=build --build-by=albin@kauff.org
```

### Création du dépôt Debian (reprepro)

J'ai utilisé *reprepro* :

```bash
mkdir -p repo/conf
vi repo/distributions
```

Dans mon cas, j'y ai écrit le contenu suivant :

```
Origin: packages.kauff.org
Label: packages.kauff.org
Suite: stable
Codename: stretch
Version: 9.0
Architectures: i386 amd64 source
Components: aseba aseba-git
Description: Packages built by Albin Kauffmann
SignWith: F3E2806C # GPG key ID
```

Ajouter les paquets à reprepro en spécifiant le composant (*aseba* ou *aseba-git* ici) :

```bash
reprepro -b repo --component aseba includedeb stretch /CHEMIN/VERS/LES/PAQUETS/*.deb
```

## Ressources

* Site Web du robot Thymio : <https://www.thymio.org>
* Spécifications du robot Thymio : <https://www.thymio.org/fr:thymiospecifications>
* Guide GPG de Nextinpact : <https://www.nextinpact.com/news/102685-gpg-comment-creer-paire-clefs-presque-parfaite.htm>
* Un autre guide GPG : <https://docs.abuledu.org/abuledu/mainteneur/creer_une_cle_gpg>
* Mini HowTo GPG : <https://www.gnupg.org/howtos/fr/GPGMiniHowto-3.html>
* Guide APT sur la signature des paquets : <https://wiki.debian.org/SecureApt>
* Création de dépôts Debian : <https://wiki.debian.org/DebianRepository/Setup#reprepro>
* Guide reprepro : <https://blog.packagecloud.io/eng/2017/03/23/create-debian-repository-reprepro>
