---
layout: post
title:  "Utiliser une vielle manette à la place d'une manette de Xbox 360"
author: Albin Kauffmann
---

<div markdown="1">

![Manette Trust]({{ "/assets/images/2020-07-05-manette-trust.png" | relative_url }})

* TOC
{:toc}

</div>{: .float-right }

Lors du confinement, j'ai ressorti une vielle manette des cartons.
Pas si mal, la bête dispose de (voir photo à droite):
- 2 axes x, y (avec un stick et un volant permettant de moduler l'amplitude de 0 à 255)
- 10 boutons (valeur 0 et 1, 0 pour relâché et 1 pour pressé)

Elle fonctionne parfaitement sous Linux mais j'ai été confronté à des soucis de compatibilité avec des jeux créés pour utiliser une manette de Xbox 360.
En effet, certains boutons était inutilisables, d'autres n'étaient pas au même endroit que sur une manette de Xbox.
J'ai donc cherché des solutions pour changer le "mapping" des touches et ainsi changer le rôle des boutons.

Mon objectif était de jouer aux jeux présents en Cloud Gaming sur [Blacknut](https://www.blacknut.com/fr) mais les techniques que je vais évoquer sont sans doute valables pour d'autres scénarios tels que le souhait de jouer à des jeux obtenus sur [Steam](https://fr.wikipedia.org/wiki/Steam).

Il m'a fallu envisager 3 techniques différentes afin de parvenir à mes fins.
Je vous propose ici une série de 3 articles dans lesquels j'évoque :
1. L'utilisation de `controllermap` pour un jeu SDL ([article ci-dessous](#configuration-sdl))
1. Le mapping des touches avec `systemd-udev` (article à venir)
1. L'émulation d'une manette en espace utilisateur avec `ubox360` (article à venir)

Après un aperçu de la manette Xbox 360, voici la première technique utilisant `controllermap`.

## Aperçu de la manette de Xbox 360

La manette de Xbox360 est composée de :
- 2 "sticks" (reconnus comme 4 axes)
- 1 "pad" directionnel (reconnu comme 2 axes)
- 9 boutons
- 2 "triggers" (reconnus comme 2 axes)

[![Xbox 360 controller](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/360_controller.svg/512px-360_controller.svg.png)](https://commons.wikimedia.org/wiki/File:360_controller.svg)

Les 2 "triggers" sont en fait reconnus comme 2 axes par le système d'exploitation (Linux dans mon cas).
Sur un jeux de voiture, cela permet de moduler la force de l'accélération ou du freinage.

## Configuration SDL

### En bref

Pour un jeux développé avec la bibliothèque SDL, il est assez simple de configurer les mappings d'une manette.

Pour cela, il faut exécuter le programme `controllermap`:

![controllermap]({{ "/assets/images/2020-07-05-controllermap-screenshot.png" | relative_url }})

Cet utilitaire affiche à l'écran les boutons d'une manette Xbox 360 et demande à l'utilisateur de presser le bouton à faire correspondre à chaque touche.
Il génère ensuite un fichier `gamecontrollerdb.txt` à placer dans le répertoire du jeux et le tour est joué !

### Installation de "controllermap"

Les utilisateurs d'ArchLinux peuvent installer facilement controllermap depuis [AUR](https://aur.archlinux.org/packages/controllermap).
Pour les autres, il faudra compiler le fichier [controllermap.c](http://hg.libsdl.org/SDL/raw-file/tip/test/controllermap.c).

Une fois installé, il faut lancer la commande suivante pour rediriger la sortie dans le fichier `gamecontrollerdb.txt`:

```bash
controllermap 0 > gamecontrollerdb.txt
```

## Conclusion

[Blacknut](https://www.blacknut.com/fr) semble être basé sur SDL mais je ne suis pas parvenu à utiliser cette méthode.
Peut-être que la gestion des manettes n'utilise pas l'API de la bibliothèque SDL ? Difficile de savoir ...

Bref, j'ai donc tenté une autre technique basée sur `systemd-udev` (article à venir).

## Ressources (en anglais)

- Page Wikipedia de la manette Xbox360 : <https://en.wikipedia.org/wiki/Xbox_360_controller>
- Page ArchLinux pour la manettes : <https://wiki.archlinux.org/index.php/Gamepad>
- Informations sur l'API SDL "GameController and Joystick Mapping" : <https://wiki.libsdl.org/CategoryGameController>
