---
layout: post
title:  "Lancer un processus avec des droits particuliers"
author: Albin Kauffmann
---

Profitons de ces jours de confinement pour reprendre un peu mon blog :-)

## Que cherche-t-on à faire ici ?

Il est parfois nécessaire d'exécuter des programmes nécessitant des droits système (accès à `/dev`, appels système particuliers, ...).
La solution la plus simple consiste évidemment à lancer ces programmes avec l'utilisateur `root` mais cela peut avoir plusieurs inconvénients :

- donner trop de droits à un programme peut amener des soucis de sécurité ou de stabilité
- l'environnement d'exécution change (valeur de `$HOME`, uid/gid des fichiers créés par le processus, etc ...)

Cet article évoque deux outils en ligne de commande permettant d'exécuter un programme en tant qu'utilisateur non privilégié (pas l'utilisateur `root` donc) mais en donnant des droits supplémentaires.

## runuser

`runuser` est un outil qui permet d'exécuter un programme en tant qu'un utilisateur et un groupe particulier.
Par exemple, voici un moyen d'exécuter un programme qui nécessite d'accéder aux périphériques d'entrée (`input`) présent dans `/dev`:

```bash
shell> sudo runuser --user=bob --group=users --supp-group=input -- mon_programme
```

Ici :

- `mon_programme` est exécuté avec l'utilisateur "bob", le groupe "users" et le groupe supplémentaire "input".
- `mon_programme` peut donc ouvrir et lire les fichiers dans `/dev/input`
-  en tout premier lieu, `runuser` est exécuté en tant que `root` (via `sudo`) afin de pouvoir affecter les uid/gid du processus `mon_programme`

Un moyen simple de tester la bonne prise en compte des arguments et d'exécuter la commande `id`:

```bash
shell> sudo runuser --user=bob --group=users --supp-group=input -- id
uid=1000(bob) gid=100(users) groups=100(users),97(input)
```

## capsh

Un autre besoin peut s'avérer nécessaire : celui de pouvoir exécuter un programme qui nécessite des capacités ([`capabilities`](https://reposcope.com/man/fr/7/capabilities)).
Supposons donc que nous souhaitions exécuter un programme nécessitant d'écouter des connexions entrantes sur un port privilégié (numéro de port inférieur à 1024).
Ce programme nécessite alors d'être exécuté avec la capacité `CAP_NET_BIND_SERVICE`:

```bash
shell> sudo capsh --user=bob --inh=cap_net_bind_service --addamb=cap_net_bind_service -- -c 'mon_programme'
```

Ici :

- l'option `--user=bob` implique l'utilisation de l'uid, du gid et des groupes supplémentaires de l'utilisateur "bob"
- les options `--inh=cap_net_bind_service` et `--addamb=cap_net_bind_service` ajoutent `CAP_NET_BIND_SERVICE` aux capacités effectives et héritables du processus `bash` exécuté par `capsh`
- l'option `-c` est l'option de `bash` qui indique que la commande entre guillemets est à exécuter

Un moyen simple de tester la bonne prise en compte des arguments et d'exécuter la commande `capsh --print`:

```bash
shell>  sudo capsh --user=bob --inh=cap_net_bind_service --addamb=cap_net_bind_service -- -c 'capsh --print'
Current: = cap_net_bind_service+eip
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Ambient set =cap_net_bind_service
Current IAB: ^cap_net_bind_service
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
 secure-no-ambient-raise: no (unlocked)
uid=1000(bob) euid=1000(bob)
gid=100(100)
groups=10(wheel),90(network),91(video),100(users)
Guessed mode: UNCERTAIN (0)
```

Enfin, voici un autre exemple d'utilisation de `capsh` qui ouvre un proxy TCP sur le port 80 afin de rediriger les requêtes HTTP sur un serveur distant :

```bash
shell> sudo capsh --user=bob --inh=cap_net_bind_service --addamb=cap_net_bind_service -- -c 'ssh -NL localhost:80:remote:80 gateway'
```

## Conclusion

Les deux outils vus permettent de donner des droits supplémentaires à des programmes sans modifier leurs codes sources.
Cela n'a pas été abordé dans cet article mais ils permettent également de restreindre les droits sur le système de fichier ainsi que les capacités.
Pour plus d'information, se référer aux pages de manuel ou aux liens fournis dans les ressources ci-dessous.

## Ressources

- Manuel des capacités Linux (capabilites) :
  - anglais : <https://www.mankier.com/7/capabilities>
  - français : <https://reposcope.com/man/fr/7/capabilities>
- Manuel de `runuser` :
  - anglais : <https://www.mankier.com/1/runuser>
  - français : <https://reposcope.com/man/fr/1/runuser>
- Manuel de `capsh` (anglais) : <https://www.mankier.com/1/capsh>
- Liens stackoverflow (anglais) :
  - <https://stackoverflow.com/questions/413807/is-there-a-way-for-non-root-processes-to-bind-to-privileged-ports-on-linux>
  - <https://stackoverflow.com/questions/28811823/using-capsh-to-drop-all-capabilities>
