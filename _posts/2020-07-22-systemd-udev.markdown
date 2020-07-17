---
layout: post
title:  "Réassocier les touches d'un clavier avec systemd-udev"
author: Albin Kauffmann
---

<div markdown="1">

* TOC
{:toc}

</div>{: .float-right }

Voici la suite de ma série d'articles visant à utiliser une vielle manette à la place d'une manette de Xbox 360 (sous un PC Linux).

Pour rappel, il m'a fallu envisager 3 techniques pour parvenir à mes fins :
1. L'utilisation de `controllermap` pour un jeu SDL ([voir article](../../../2020/07/05/sdl-controllermap.html))
1. Le mapping des touches avec `systemd-udev` ([article ci-dessous](#mapping-des-touches-avec-systemd-udev))
1. L'émulation d'une manette en espace utilisateur avec `ubox360` (article à venir)

Je propose ci-dessous une méthode permettant de changer l'association des touches d'une manette de jeu.
Cette méthode, appelée "mapping" en anglais, peut également fonctionner pour n'importe quel type de périphérique d'entrée (clavier, souris, ...).

## Présentation

Il est possible de mapper les touches d'un périphérique d'entrée avec systemd-udev (utilisé dans la plupart des distributions récentes).

Cela fonctionne :
- pour n'importe quel type de périphérique d'entrée (manette, clavier, souris, ...)
- pour n'importe quelles applications (SDL, openGL, ...) car c'est une configuration bas-niveau

Voici tout d'abord comment lister les capacités d'une manette, puis comment créer cette configuration systemd-udev.

## Lister les capacités d'une manette

Considérons pour les commandes qui suivent que le branchement de ma manette de jeux crée le périphérique `/dev/input/event11` ("input device").
On peut généralement trouver ce chemin en listant les liens symboliques présents dans `/dev/input/by-id` :

```bash
shell> ls -l /dev/input/by-id/usb-0458_100a-event-joystick
lrwxrwxrwx 1 root root 10 Jun 29 22:24 /dev/input/by-id/usb-0458_100a-event-joystick -> ../event11
```

Pour en savoir plus sur le périphérique (product ID, vendor ID, ...), il est possible d'afficher le contenu des fichiers contenus dans `/sys/class/input/event11/device/id/` :

```bash
shell> ls -la /sys/class/input/event11/device/id/
bustype  product  vendor  version
```

Pour afficher les événements du périphérique, il faut utiliser `evtest` :

```bash
shell> evtest /dev/input/event11
Input driver version is 1.0.1
Input device ID: bus 0x3 vendor 0x458 product 0x100a version 0x100
Input device name: "HID 0458:100a"
Supported events:
  Event type 0 (EV_SYN)
  Event type 1 (EV_KEY)
    Event code 304 (BTN_SOUTH)
    Event code 305 (BTN_EAST)
    Event code 306 (BTN_C)
    Event code 307 (BTN_NORTH)
    Event code 308 (BTN_WEST)
    Event code 309 (BTN_Z)
    Event code 310 (BTN_TL)
    Event code 311 (BTN_TR)
    Event code 312 (BTN_TL2)
    Event code 313 (BTN_TR2)
  Event type 3 (EV_ABS)
    Event code 0 (ABS_X)
      Value    129
      Min        0
      Max      255
      Flat      15
    Event code 1 (ABS_Y)
      Value    129
      Min        0
      Max      255
      Flat      15
  Event type 4 (EV_MSC)
    Event code 4 (MSC_SCAN)
Properties:
Testing ... (interrupt to exit)
Event: time 1593462939.076514, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90001
Event: time 1593462939.076514, type 1 (EV_KEY), code 304 (BTN_SOUTH), value 1
Event: time 1593462939.076514, -------------- SYN_REPORT ------------
Event: time 1593462939.196331, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90001
Event: time 1593462939.196331, type 1 (EV_KEY), code 304 (BTN_SOUTH), value 0
Event: time 1593462939.196331, -------------- SYN_REPORT ------------
Event: time 1593462940.100397, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90002
Event: time 1593462940.100397, type 1 (EV_KEY), code 305 (BTN_EAST), value 1
Event: time 1593462940.100397, -------------- SYN_REPORT ------------
Event: time 1593462940.172482, type 4 (EV_MSC), code 4 (MSC_SCAN), value 90002
Event: time 1593462940.172482, type 1 (EV_KEY), code 305 (BTN_EAST), value 0
Event: time 1593462940.172482, -------------- SYN_REPORT ------------
```

On y aperçoit que la manette supporte :
- 2 axes ABS_X et ABS_Y
- 10 boutons (EV_KEY)

Et lorsque l'on presse les boutons, `evtest` affiche notamment :
- l'`event code` envoyé à l'application utilisant la manette (304 = BTN_SOUTH, 305 = BTN_EAST, ...) dont les valeurs possibles se trouvent dans le fichier en-tête `/usr/include/linux/input-event-codes.h`)
- le `scan code` (90001, 90002, ...) qui est le code envoyé par le périphérique

## Créer la configuration systemd-udev

Commencer par créer le fichier mapping suivant `/etc/udev/hwdb.d/70-keyboard.hwdb` :

```
evdev:input:b0003v0458p100Ae0100-*
 KEYBOARD_KEY_90001=btn_a
 KEYBOARD_KEY_90002=btn_b
 KEYBOARD_KEY_90003=btn_select
 KEYBOARD_KEY_90004=btn_x
 KEYBOARD_KEY_90005=btn_y
 KEYBOARD_KEY_90006=btn_start
 KEYBOARD_KEY_90007=btn_tl
 KEYBOARD_KEY_90008=btn_tr
 KEYBOARD_KEY_90009=btn_trigger_happy1
 KEYBOARD_KEY_9000a=btn_trigger_happy2

```

Explications :
- `b0003v0458p100Ae0100` identifie mon périphérique (cf valeurs présentes dans les fichiers de `/sys/class/input/event11/device/id/`) :
  - `b0003` indique que c'est de l'USB
  - `v0458` indique l'ID vendeur
  - `p100A` indique l'ID produit
  - `e0100` indique la version
- les valeurs `90001` à `9000a` sont les "scan codes" des boutons que je souhaite "mapper"
- les chaînes de caractère btn_a, btn_b, ... sont les "event codes" que je souhaite affecter aux boutons (cf `/usr/include/linux/input-event-codes.h` pour la liste des codes existants)

Le nom du fichier `70-keyboard.hwdb` est important.
Consultez les commentaires du fichier `/usr/lib/udev/hwdb.d/60-keyboard.hwdb` pour plus d'informations.

Enfin, il faut recharcher la configuration systemd-udev avec les deux commandes suivants :

```bash
# Création de la base de données de mappings (fichier /dev/udev/hwdb.bin)
shell> systemd-hwdb update
# Application des nouveaux bindings au périphérique (évite de devoir rebrancher la manette)
shell> udevadm trigger /dev/input/event11
# Vérification de la bonne prise en compte des bindings
shell> evtest /dev/input/event11
[...]
```

## Conclusion

Cette méthode fonctionne parfaitement pour changer la position des boutons.
Mais les 2 boutons "trigger" de la manette Xbox sont en fait des axes (`ABS_Z` et `ABS_RZ`) alors que sur ma manette ce sont des boutons (`BTN_*`).

J'aimerais transformer ces 2 boutons en axes à 2 valeurs, 0 et 1 par exemple, sans aucune variation possible de cette valeur (suffisant pour certains jeux dans lesquels il n'est pas nécessaire de moduler la pression sur les triggers).

Cependant `systemd-udev` ne permet pas de changer le type d'événement (un appui sur un bouton ne peut pas devenir un évènement sur un axe).
J'ai donc mis au point une autre technique basée sur un programme personnel `ubox360` (article à venir).

## Ressources (en anglais)

- Page Wiki ArchLinux pour mapper les touches : <https://wiki.archlinux.org/index.php/Map_scancodes_to_keycodes>
- Page Wiki ArchLinux pour identifier les touches : <https://wiki.archlinux.org/index.php/Keyboard_input#Identifying_scancodes>
- Présentation des touches de différentes manettes : <https://pineight.com/mw/index.php?title=USB_game_controller>
- Liste des keycodes d'une manette Xbox : <https://www.linux.org/threads/xbox-360-controller.11422/>
