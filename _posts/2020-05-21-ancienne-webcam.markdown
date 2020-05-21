---
layout: post
title:  "Utiliser une ancienne webcam V4L1 pour de la vidéoconférence"
author: Albin Kauffmann
---

<div markdown="1">

* TOC
{:toc}

![Kocom KDC-130]({{ "/assets/images/2020-05-21-webcam-kdc-130.jpg" | relative_url }})

</div>{: .float-right }

Confinement oblige, j'ai ressorti du vieux matériel pour conserver une vie sociale.
Voici donc (en photo à droite) ma webcam des années 2000, capable de fournir une magnifique résolution de 320x242 avec 15 images par secondes (trop fou :D).

Problème, elle n'est pas compatible avec la version actuelle de Video4Linux (V4L), la couche logicielle de Linux qui gère la capture de flux vidéo.
Le noyau Linux utilise exclusivement la version V4L2 et l'utilisation de V4L1 est uniquement possible en espace utilisateur ("user space").

Cet article propose donc une configuration permettant l'utilisation d'une caméra V4L1 avec un logiciel de vidéoconférence tel que Microsoft Teams ou même Firefox (pour Jitsi Meet) dans mon cas.

Il y 2 étapes :
- utiliser ffmpeg pour récupérer le flux vidéo V4L1
- écrire le flux vidéo dans un périphérique virtuel de type V4L2 (lisible par n'importe quel application de vidéoconférence)

## Lecture du flux V4L1

Afin de lire un flux V4L1, il faut simplement charger la bibliothèque de compatibilité `v4l1compat.so` grâce à la variable d'environnement `LD_PRELOAD`.
Celle-ci devrait être fournie dans le paquet `v4l-utils` de votre distribution Linux.

Commençons par lister les possibilités de notre caméra (dont le périphérique est `/dev/video0`) :

```bash
shell> LD_PRELOAD=/usr/lib/libv4l/v4l1compat.so ffmpeg -f v4l2 -list_formats all -i /dev/video0
[...]
[video4linux2,v4l2 @ 0x5611adfa0240] Raw       :       rgb24 :                 RGB3 : Emulated : 320x242
[video4linux2,v4l2 @ 0x5611adfa0240] Raw       :       bgr24 :                 BGR3 : Emulated : 320x242
[video4linux2,v4l2 @ 0x5611adfa0240] Raw       :     yuv420p :                 YU12 : Emulated : 320x242
[video4linux2,v4l2 @ 0x5611adfa0240] Raw       :     yuv420p :                 YV12 : Emulated : 320x242
```

Après avoir choisi une résolution (si vous avez le choix), il est possible de lire le flux vidéo et de l'afficher avec ffplay :

```bash
LD_PRELOAD=/usr/lib/libv4l/v4l1compat.so ffmpeg -f v4l2 -video_size 320x242 -i /dev/video0 -f rawvideo - | ffplay -video_size 320x242 -f rawvideo -
```

## Génération du flux V4L2

L'idée ici est de créer un périphérique virtuel `/dev/video1` utilisable par n'importe quel logiciel de vidéoconférence.
Il faut donc installer [v4l2loopback](https://github.com/umlaeute/v4l2loopback) (disponible via AUR pour les utilisateurs d'ArchLinux).

Une fois le module noyau chargé, il suffit d'exécuter la commande suivante :

```bash
LD_PRELOAD=/usr/lib/libv4l/v4l1compat.so ffmpeg -f v4l2 -i /dev/video0 -f v4l2 /dev/video1
```

Lancer ensuite votre vidéoconférence en utilisant la caméra virtuelle `/dev/video1`.

## Ressources (en anglais)

- Page Wiki de ArchLinux (section V4L1): <https://wiki.archlinux.org/index.php/Webcam_setup#V4L1_support>
- Sources du pilote v4l2loopback: <https://github.com/umlaeute/v4l2loopback>
