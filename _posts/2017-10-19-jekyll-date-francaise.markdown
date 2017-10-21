---
layout: post
title:  "Afficher les dates en français avec Jekyll"
author: Albin Kauffmann
categories: jekyll
---

Une installation [Jekyll](http://jekyllrb.com) de base ne supporte pas la
localisation de la date, c'est à dire que Jekyll n'offre pas la possibilité
d'afficher une date au format français.

En effet, par défaut Jekyll va afficher la date des articles avec le format
suivant :

    {{ page.date | date: "%b %-d, %Y" }}

Et nous souhaiterions par exemple obtenir son équivalent en français soit :

    {% assign date_english = page.date | date: "%-d %b %Y" %}
    {% include date-french.html %}
    {{ date_french }}

J'ai trouvé 3 solutions pour résoudre ce soucis, la dernière étant la seule
compatible avec les [pages GitHub](https://pages.github.com).
Les voici :

## Utilisation d'un plugin Jekyll

Des plugins Jekyll permettent de créer un site multilingue.
Ces plugins ne sont pas disponibles avec les pages GitHub (voir la liste des
plugins supportés [ici](https://pages.github.com/versions)) et ils me semblent
trop complets pour mon besoin.
Je n'ai donc pas testé cette solution.

Les 3 plugins que j'ai trouvé 3 pour créer un site multilingue sont les
suivants :

* [Jekyll Language Plugin](https://github.com/vwochnik/jekyll-language-plugin)
* [Jekyll i18n](https://github.com/liamzebedee/jekyll-i18n)
* [Jekyll Multiple Languages Plugin](https://github.com/Anthony-Gaudino/jekyll-multiple-languages-plugin)

## Développement d'un plugin Jekyll

Un solution plus simple consiste à implémenter un plugin Jekyll de type
« [Liquid filter](http://jekyllrb.com/docs/plugins/#liquid-filters) » qui
utilise le format, les jours et les mois français de la date.
Celui-ci est à intégrer dans l'arborescence du site Jekyll, dans le répertoire
« _plugins ».

Voici un
[exemple de code](https://github.com/albinou/albinou.github.io/blob/3e74deacab33d31e15dae603e8ba3454d4bd997f/_plugins/date_filter_french.rb)
disponible sur mon GitHub (et sans doute améliorable).
Ce plugin exporte une fonction « _date_to_french_ » réutilisable dans le code
Liquid des pages (tel que celles dans le répertoire __layout_).

L'utilisation de ce plugin n'est donc pas automatique.
Il faut modifier les pages qui appellent le filtre « date ».
La modification de la page _home.html_ du thème Minima ressemble à ceci :

{% raw %}
```diff
-{% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
-<span class="post-meta">{{ post.date | date: date_format }}</span>
+<span class="post-meta">{{ post.date | date_to_french }}</span>
```
{% endraw %}

GitHub génère les sites web avec l'option `--safe` de Jekyll, ce qui a pour
conséquence de désactiver les plugins personnalisés. Cette solution n'est donc
pas compatible avec la génération de pages de GitHub.

## Sans plugin

Si cette solution est la moins élégante, elle a le mérite de fonctionner avec la
génération de pages de GitHub.
Mon idée a été de remplacer le jour et le mois de la date générée par le filtre
« _date_ » de Liquid.
On conserve ainsi le choix du format de date spécifié dans le fichier
__config.yml_ de Jekyll (variable _minima.date_format_ dans le cas du thème
Minima).

J'ai placé le code Liquid dans un fichier
[_include/date-french.html](https://github.com/albinou/albinou.github.io/blob/3b08f660608b76e8ff4ea8525ec552ec5db49d3c/_includes/date-french.html)
ensuite inclus par les pages qui nécessite l'affichage d'une date.
La modification de la page
[home.html](https://github.com/albinou/albinou.github.io/blob/3b08f660608b76e8ff4ea8525ec552ec5db49d3c/_layouts/home.html)
du thème Minima ressemble à ceci :

{% raw %}
```diff
 {% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
-<span class="post-meta">{{ post.date | date: date_format }}</span>
+{% assign date_english = post.date | date: date_format %}
+{% include date-french.html %}
+<span class="post-meta">{{ date_french }}</span>
```
{% endraw %}

## Ressources

* Les plugins _Liquid filters_ de Jekyll : <http://jekyllrb.com/docs/plugins/#liquid-filters>
* Documentation sur le langage _Liquid_ : <https://github.com/Shopify/liquid/wiki/Liquid-for-Designers>
* Exemple de formatage de la date avec _Liquid_ : <http://alanwsmith.com/jekyll-liquid-date-formatting-examples>
* Manuel du programmeur pour les formats de dates : <http://man7.org/linux/man-pages/man3/strftime.3.html>
