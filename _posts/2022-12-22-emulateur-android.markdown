---
layout: post
title:  "Lancer l'émulateur Android sous ArchLinux"
author: Albin Kauffmann
---

<div markdown="1">

* TOC
{:toc}

</div>{: .float-right }

Il peut s'avérer très pratique d'exécuter une application Android sur son ordinateur.
Ceci pour des raisons très variables comme par exemple pour :

- exécuter une application sans l'installer sur son propre ordiphone
- tester une application sur une version spécifique d'Android

Il y a sans doute milles méthodes de faire.
Voici comment je procède sur mon PC sous ArchLinux (en utilisant les paquets de [AUR](https://wiki.archlinux.org/title/Arch_User_Repository)).
Avec cette procédure, les SDK sont téléchargés en tant que "root" puis les périphériques Android (les "AVDs" pour "Android Virtual Devices") sont créés avec un utilisateur standard.

## Installation des paquets AUR

Il est nécessaire d'installer les paquets suivants :

- android-emulator (fourni la commande `emulator`)
- android-sdk-cmdline-tools-latest (fourni les commandes `sdkmanager` et `avdmanager`)
- android-sdk-platform-tools (nécessaire à `emulator` et fourni des commandes tels que `adb`)

Par exemple, avec [pikaur](https://aur.archlinux.org/packages/pikaur), l'installation se fait avec la commande suivante :

```shell
pikaur -S android-emulator android-sdk-cmdline-tools-latest android-sdk-platform-tools
```

## Téléchargement du SDK et de l'image système

Certains SDK et certaines images systèmes sont disponibles dans AUR mais les dernières versions ne semblent pas y êtres présentes.
Je propose donc ici d'utiliser la commande `sdkmanager` de Google afin de télécharger ces éléments.
Voici les différentes étapes :

1. vérifier votre [umask](https://wiki.archlinux.org/title/Umask).
  En effet, les commandes exécutées dans les étapes suivantes vont créer des fichiers dans `/opt/android-sdk` et ces fichiers doivent être créés avec les droits de lecture pour "other (o)".
  Du coup, je conseille le umask suivant :

    ```shell
    umask 022
    ```

1. choisir un SDK et une image système en listant les paquets disponibles à l'aide de la commande suivante :

    ```shell
    sdkmanager --list
    ```

    Noter alors le "path" de la version qui vous intéresse.
    Dans la suite de cet article, j'ai sélectionné Android Pie (SDK version 28) en raison de sa taille raisonnable et du fait que cette version devrait suffire dans 98% des cas.

1. installer le SDK (ici la version 28) :

    ```shell
    sudo sdkmanager 'platforms;android-28'
    ```

1. installer une image système correspondant à la version du SDK précédemment installé (ici une image d'Android Pie incluant le Google Play store) :

    ```shell
    sudo sdkmanager 'system-images;android-28;google_apis_playstore;x86_64'
    ```

1. vérifier que les paquets ont correctement été installés :

    ```shell
    sdkmanager --list_installed
    ```

## Création du périphérique Android "AVD"

Cette création se fait sans le compte "root" cette fois-ci, et avec un compte utilisateur classique :

```shell
avdmanager create avd --name "Android Pie" --package 'system-images;android-33;google_apis;x86_64'
```

Pour information, cette commande crée des fichiers dans `$HOME/.android/avd`.
Supprimer un AVD pourra se faire avec la commande suivante :

```shell
avdmanager delete avd -n "Android Pie"
```

## Lancer l'émulateur (enfin !)

Il suffit de lancer la commande suivante :

```shell
emulator -avd "Android Pie"
```

Android est assez gourmand en mémoire.
Il est cependant possible de limiter son utilisation mémoire avec l'option `-memory` comme ci-dessous :

```shell
emulator -avd "Android Pie" -memory 1024
```

## Ressources

* Documentation d'`sdkmanager`: <https://developer.android.com/studio/command-line/sdkmanager>
* Documentation d'`avdmanager`: <https://developer.android.com/studio/command-line/avdmanager>
* Documentation d'`emulator`: <https://developer.android.com/studio/run/emulator-commandline>
