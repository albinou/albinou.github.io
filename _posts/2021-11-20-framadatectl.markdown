---
layout: post
title:  "Automatisation de la gestion d'un Framadate"
author: Albin Kauffmann
---

<div markdown="1">

* TOC
{:toc}

</div>{: .float-right }

Dans le cadre de la gestion des distributions (ou « partages de récoltes ») d'une [AMAP](http://www.amap-idf.org/qu_est-ce_qu_une_amap_176.php), j'ai ressenti le besoin d'automatiser la gestion d'un sondage [Framadate](https://framadate.org).

Nos distributions ont lieu tous les mercredis et nécessitent 3 volontaires qui s'inscrivent auparavant sur un Framadate comme celui-ci:

[![Framadate]({{ "/assets/images/2021-11-20-framadate-screenshot.png" | relative_url }}){: width="75%" }](
              {{ "/assets/images/2021-11-20-framadate-screenshot.png" | relative_url }})

Pour me libérer d'une partie de la charge mentale liée à l'organisation de ces distributions hebdomadaires, j'ai développé un outil nommé [framadatectl](https://gitlab.com/albinou/python-framadatectl) (pour "Framadate controller").
[framadatectl](https://gitlab.com/albinou/python-framadatectl) permet d'automatiser les tâches suivantes (mes "jobs") :
- **le lundi :** si moins de 3 distributeur·rice·s sont inscrit·e·s à la distribution du mercredi, un email d'avertissement est envoyé à tou·te·s les adhérent·e·s
- **le mardi :** la liste définitive des distributeur·rice·s est envoyée au responsable de la distribution
- **le mercredi :** rien, mais on récupère nos super légumes 🥕🍅 !
- **le jeudi :** un petit ménage du Framadate est effectué :
  - sauvegarde des distributeurs de la veille
  - suppression de la date de la veille
  - suppression des votes vides (fréquent, puisqu'une date vient d'être supprimée)

Mais [framadatectl](https://gitlab.com/albinou/python-framadatectl) n'est pas uniquement orienté AMAP puisque :
1. il permet d'interagir avec un Framadate en ligne de commande
1. il utilise un fichier de configuration assez poussé (et qui évoluera sans doute encore), ce qui le rend réutilisable pour d'autres choses
1. il peut aussi être utilisé en tant que bibliothèque Python (cet article ne détaillera pas l'API, mais il est possible de consulter la documentation de cette bibliothèque à [l'adresse suivante](https://gitlab.com/albinou/python-framadatectl#library))

## Installation de framadatectl

### Pré-requis

[framadatectl](https://gitlab.com/albinou/python-framadatectl) requiert :
- Python 3.7 minimum (il a été développé pour Linux mais devrait pouvoir également fonctionner sous Windows)
- un serveur avec une table de planification ou [cron](https://fr.wikipedia.org/wiki/Cron) (afin d'exécuter des actions de manière cyclique, hebdomadaire par exemple)

### Installation

Le plus simple est d'installer [framadatectl](https://gitlab.com/albinou/python-framadatectl) à l'aide de `pip` (ou `pip3` sur certaines distributions Linux) :

```bash
pip install framadatectl
```

### Utilisation basique

Après avoir remplacé `https://framadate.org/IDENTIFIER` par le lien public (ou le lien d'administration) d'un sondage, la commande suivante retourne les résultats du sondage :

```
framadatectl --url https://framadate.org/IDENTIFIER show all-slots
2021-09-01: [Bob, Alice, John] (3 vote(s))
2021-09-08: [Bob, Alice] (2 vote(s))
2021-09-15: [John] (1 vote(s))
```

Pour des commandes plus avancées, ne pas hésiter à utiliser l'aide (option `--help`).

### Configuration basique

Pour utiliser les fonctions avancées de [framadatectl](https://gitlab.com/albinou/python-framadatectl), il est nécessaire de lui donner de nombreuses options de configuration.
Il est donc plus simple de passer ces informations par un fichier de configuration au format [YAML](https://fr.wikipedia.org/wiki/YAML).

Voici un exemple de fichier de configuration basique `config.yaml` :

```yaml
configuration:
  url: https://framadate.org/IDENTIFIER
  constraints:
    votes:
      moment_regexp: '^(\d+) dist\..*$'
```

Cette configuration indique :
- l'adresse (`url`) du sondage
- une [expression régulière](https://fr.wikipedia.org/wiki/Expression_régulière) (`moment_regexp`)

`moment_regexp` indique à [framadatectl](https://gitlab.com/albinou/python-framadatectl) la manière de déterminer le nombre de votes nécessaires en se basant sur le champs "Horaire" de Framadate (cf. capture d'écran en haut de la page).
Par exemple :
- "3 dist. légumes" indique que 3 distributeur·rice·s sont nécessaires pour distribuer les légumes
- "1 dist. pommes" indique que 1 distributeur·rice est nécessaire pour distribuer les pommes

Avec ce fichier de configuration, il est ainsi possible de vérifier le statut des distributions :

```
framadatectl --config config.yaml check all-slots
2021-11-24 (3 dist. légumes): OK (3 vote(s))
2021-12-01 (3 dist. légumes): KO (1 missing vote(s))
2021-12-08 (1 dist. pommes): OK (1 vote(s))
2021-12-08 (3 dist. légumes): KO (3 missing vote(s))
2021-12-15 (3 dist. légumes): KO (3 missing vote(s))
2021-12-22 (3 dist. légumes): KO (3 missing vote(s))
```

À noter qu'il est possible d'indiquer un minimum (`min`) dans le fichier de configuration et de ne pas se baser sur `moment_regexp`.

## Définir et utiliser des "jobs"

### Configuration des "jobs"

La configuration des "jobs" se fait dans le fichier de configuration.
Chaque "job" nécessite :
- un `id` : nom donné au job (permettra de le lancer)
- une `condition` : tableau (potentiellement vide) de conditions
- une `action` : tableau d'actions à exécuter si toutes les conditions précédemment listées sont vérifiées

Voici par exemple la configuration que j'utilise pour mon AMAP et qui définit 3 "jobs" (`warn`, `info` et `clean`) :

```yaml
configuration:
  url: https://framadate.org/IDENTIFIER/admin
  constraints:
    votes:
      moment_regexp: '^(\d+) dist\..*$'
  email:
    smtp_host: localhost
job:
  - id: warn
    # Vérifie si suffisamment d'adhérent·e·s sont inscrit·e·s pour la prochaine
    # distribution. Envoie un email si ce n'est pas le cas.
    condition:
      - condition: slots
        slots: next-date
        verify:
          constraints: True
    action:
      - action: email
        slots: next-date
        content: |
          From: Robot AMAP <robot@example.org>
          To: adherents@mailinglist.example.org
          Content-Type: text/plain; charset=utf-8
          Content-Language: fr-FR
          Subject: Besoin de distributeurs pour ce {date:%A %d %B}
          
          Bonjour à tous,
          
          Il manque {missing_votes} amapien·ne(s) pour la distribution de ce {date:%A %d %B}.
          Pour s'inscrire, c'est ici : https://framadate.org/PUBLIC_IDENTIFIER
          
          À bientôt !
          
          --
          Ce message a été généré par un gentil robot.
  - id: info
    # Envoie la liste des distributeur·rice·s de la prochaine distribution.
    condition: []
    action:
      - action: email
        slots: next-date
        content: |
          From: Robot AMAP <robot@example.org>
          To: referant@example.org
          Content-Type: text/plain; charset=utf-8
          Content-Language: fr-FR
          Subject: [AMAP] Distributeurs de ce {date:%A %d %B}
          
          Hello,
          
          Pour ce {date:%A %d %B}, il y a {nb_votes} distributeur·rice·s : {votes}.
          
          --
          Ce message a été généré par un gentil robot.
  - id: clean
    # Sauvegarde les ancien·ne·s distributeur·rice·s et supprime les anciennes
    # dates.
    condition: []
    action:
      - action: backup
        slots: old-slots
        mode: a
        filepath: /PATH/TO/backup_votes.txt
      - action: command
        command: delete
        subcommand: old-slots
      - action: command
        command: delete
        subcommand: empty-votes
```

Pour plus d'informations sur la configuration, consulter la [documentation (en anglais)](https://gitlab.com/albinou/python-framadatectl#running-jobs).

### Lancer un "job"

Voici comment, par exemple, exécuter le "job" warn définit ci-dessus :

```bash
LANG=fr_FR.UTF-8 framadatectl --config config.yaml job warn
```

`LANG=fr_FR.UTF-8` permet de correctement formater la date (envoyée dans le corps du mail) en français.

### Définir un timer avec systemd

Il existe de multiple manière de planifier un [cron](https://fr.wikipedia.org/wiki/Cron).
Je détaille ici comment faire avec systemd.

1. Créer un fichier service, par exemple `/etc/systemd/system/amap-warn.service` :
    ```
    [Unit]
    Description=Amap warning cron job
    
    [Service]
    User=albinou
    Group=albinou
    WorkingDirectory=/path/to/workingdir
    Environment="LANG=fr_FR.UTF-8"
    ExecStart=/usr/local/bin/framadatectl --config /path/to/config.yaml job warn
    
    [Install]
    WantedBy=basic.target
    ```
1. Créer un fichier timer avec le même nom (`/etc/systemd/system/amap-warn.timer`) :
    ```
    [Unit]
    Description=Run Amap warn cron job every Monday at 11:42 am
    
    [Timer]
    Unit=amap-warn.service
    OnCalendar=Mon *-*-* 11:42:42
    
    [Install]
    WantedBy=timers.target
    ```
1. Activer le timer :
    ```bash
	sudo systemctl enable amap-warn.timer
	sudo systemctl start amap-warn.timer
	```
1. Faire la même chose avec tous vos "jobs"

## Et maintenant ?

[framadatectl](https://gitlab.com/albinou/python-framadatectl) pourrait encore évoluer.
J'ai notamment en tête :
- ajout de la configuration du serveur mail SMTP (avec authentification au lieu de simplement utiliser localhost)
- synchronisation des dates du Framadate avec un calendrier (type iCal)
- ...

Quoiqu'il en soit, si vous avez besoin d'aide pour utiliser [framadatectl](https://gitlab.com/albinou/python-framadatectl), n'hésitez pas à poster des commentaires ou à [me contacter en privé](http://albin.kauff.org/contact.html).
Si c'est pour une AMAP, je vous aiderai volontier !
