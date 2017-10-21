---
layout: post
title:  "Lire une vidéo en vitesse accélérée avec Kodi"
author: Albin Kauffmann
categories: kodi
---

Afin de gagner du temps, il est pratique de pouvoir regarder certains
documentaires ou conférences en vitesse accélérée (1.5x voir même 2x).

Depuis la version 17 de Kodi, c'est possible.
En effet, des actions de contrôle du "player" ont été ajoutées :

* ```TempoUp``` (augmentation de la vitesse de lecture de 0.1s)
* ```TempoDown``` (diminution de la vitesse de lecture de 0.1s)

A priori, l'utilisation de ces commandes fonctionnent sur n'importe quel type de
flux : fichier accessible localement ou en réseau, streaming (Youtube, replay de
la chaîne Arte, ...).
Voici dans cet article comment utiliser cette fonctionnalité.

## Activation

Tout d'abord, vérifiez que vous utilisez Kodi version 17 ou supérieur (dans mon
cas, j'utilise la version 17.3 fournie par l'image 8.0.2 de
[LibreELEC](https://libreelec.tv) pour le [Wetek
Play](https://wetek.com/ww/en/product/wetek-play)).

Il faut ensuite activer l'option "Synchroniser la lecture avec l'affichage" des
paramètres du lecteur.
Pour cela :

1. Aller dans les paramètres de Kodi puis cliquer sur "Paramètres du lecteur" :
![Paramètres de Kodi]({{ "/assets/images/2017-10-23-kodi-parametres.jpg" | absolute_url }})

2. Dans la section "vidéo", cocher la case "Synchroniser la lecture avec
l'affichage"
![Paramètres du lecteur]({{ "/assets/images/2017-10-23-kodi-parametres-lecteur.jpg" | absolute_url }})

## Utilisation

### En ligne de commande

Pour tester, il est déjà possible d'utiliser les commandes suivantes afin
d'observer que l'on obtient bien le comportement attendu :

* `kodi-send --action="PlayerControl(tempoup)"`
* `kodi-send --action="PlayerControl(tempodown)"`

Ces commandes sont à exécuter lors de la lecture d'une vidéo et la vitesse de
lecture devrait alors s'afficher dans la barre de navigation de votre vidéo :

![Barre de navigation lecture vidéo]({{ "/assets/images/2017-10-23-kodi-lecture.jpg" | absolute_url }})

A noter : avec Kodi 17.3, il est possible de faire varier la vitesse de lecture
entre 0.8 et 1.5.
Après lecture du code sur le [GitHub du projet](https://github.com/xbmc/xbmc),
il me semble que cette limite disparaît avec Kodi 17.4, mais à vérifier (je n'ai
pas testé).

### Avec un clavier

Je n'ai pas testé avec un clavier mais il semble possible d'utiliser la
fonctionnalité tempo avec les touches :
* `ALT+flèche gauche` : diminution de la vitesse de lecture de 0.1s
* `ALT+flèche droite` : augmentation de la vitesse de lecture de 0.1s

En effet, le fichier de configuration par défaut de Kodi (disponible dans
"/usr/share/kodi/system/keymaps/keyboard.xml" sous LibreELEC) définit les
touches suivantes :

```xml
<keymap>
  [...]
  <FullscreenVideo>
    <keyboard>
      <left mod="alt">PlayerControl(tempodown)</left>
      <right mod="alt">PlayerControl(tempoup)</right>
    </keyboard>
  </FullscreenVideo>
  [...]
</keymap>
```

### Avec une télécommande (cas du WeTek Play)

La télécommande du Wetek Play ([version
1](https://wetek.com/ww/en/product/wetek-play)) n'a par défaut pas de touche
définie pour accélérer et ralentir la lecture d'une vidéo.

J'ai donc redéfini 2 touches jusqu'alors pas très utiles pour cet usage :

* `flèche bas` : diminution de la vitesse de lecture de 0.1s
* `flèche haut` : augmentation de la vitesse de lecture de 0.1s

Pour cela, j'ai créé un fichier "~/.kodi/userdata/keymaps/wetek-play.xml"
contenant :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<keymap>
  <FullscreenVideo>
    <keyboard>
      <down>PlayerControl(tempodown)</down>
      <up>PlayerControl(tempoup)</up>
    </keyboard>
  </FullscreenVideo>
</keymap>
```

Il faut redémarrer Kodi pour que ce fichier de configuration soit pris en
compte.

## Ressources

* Post anglais évoquant la solution sur forum de Kodi : <https://forum.kodi.tv/showthread.php?tid=10023&pid=2420219#pid2420219>
* Liste des fonctions de Kodi (API) : <http://kodi.wiki/view/List_of_built-in_functions>
* GitHub du projet Kodi : <https://github.com/xbmc/xbmc>
