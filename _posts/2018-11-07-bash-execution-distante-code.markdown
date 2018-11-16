---
layout: post
title:  "Exécuter du code shell sur une machine distante"
author: Albin Kauffmann
---

Cet article explique une technique permettant d'exécuter des fonctions shell
sur une machine distante.
Ce besoin peut se présenter pour des opérations complexes de maintenance ou
peut-être même pour du calcul partagé, qui sait !

Exécuter un programme à distance avec _ssh_ est très aisé (cf. Introduction)
mais les choses deviennent plus compliquées pour :

* transmettre des arguments complexes (tableaux et tableaux associatifs)
* récupérer des résultats complexes

La solution proposée dans cet article repose sur des éléments de syntaxe et sur
des commandes internes (_builtins_) propres à bash (version 4.4 minimum).
Il est donc fort possible qu'elle ne fonctionne pas avec d'autres interpréteurs
shell.

**Pressé(e) ?** -- Si vous êtes déjà très familier avec bash et notamment avec les
builtins _local -n_ et _typeset -p_, vous pouvez directement sauter à la
section "Exécuter une fonction bash à distance".

## Introduction

En langage shell, il est facile d'exécuter une commande via _ssh_ et de
récupérer sa sortie standard dans une variable (chaîne de caractères).
Voici un cours script _bash_ permettant de récupérer la version de Debian de la
machine _exemple.org_:

**Code bash :**

```bash
#!/bin/bash
debian_version="$(ssh exemple.org cat /etc/debian_version)"
echo $debian_version
```

**Exécution :**

```
9.5
```

Il est également envisageable d'ajouter à ce script la récupération de la
version du noyau Linux :

**Code bash :**

```bash
#!/bin/bash
debian_version="$(ssh exemple.org cat /etc/debian_version)"
echo $debian_version
kernel_version="$(ssh exemple.org uname -r)"
echo $kernel_version
```

**Exécution :**

```
9.5
4.9.0-8-amd64
```

Il serait possible d'ajouter des appels supplémentaires à _ssh_ pour chaque
nouvelle donnée à récupérer.
Mais utiliser _ssh_ de telle sorte posent plusieurs soucis :

* chaque appel à _ssh_ est coûteux et lent
* l’organisation du code est fastidieuse pour des cas complexes
* il n'est pas possible de transmettre ou récupérer des données complexes
(tableaux par exemple)

Dans cet article, nous allons donc voir comment exécuter des fonctions bash sur
une machine distante.
Pour cela, nous devons d'abord évoquer des usages avancés de bash.
C'est l'objet des deux prochaines sections.

## Obtenir la définition de variables et de fonctions (_typeset -p_)

Grâce à _typeset -p_, il est possible de faire une sorte d'introspection de code
pour afficher la définition d'une variable ou d'une fonctions.
Cette possibilité sera utilisée plus tard pour exporter des variables et des
fonctions dans une autre instance de bash.

Voici ci-dessous plusieurs exemple d'utilisation de _typeset -p_.

### Obtenir la définition d'une variable simple avec _typeset -p_

Le code suivant déclare un entier, change sa valeur puis affiche la définition
complète de cet entier :

**Code bash :**

```bash
#!/bin/bash
declare -i my_int=42
my_int=78
typeset -p my_int
```

**Exécution :**

```
declare -i my_int="78"
```

### Obtenir la définition d'un tableau avec _typeset -p_

Ceci fonctionne également pour des types de données plus complexes comme les
tableaux associatifs :

**Code bash :**

```bash
#!/bin/bash
declare -A my_colors_map=(
    ["blue"]="bleu"
    ["red"]="rouge"
    ["vert"]="green"
)
my_colors_map["white"]="blanc"
my_colors_map["black"]="noir"
typeset -p my_colors_map
```

**Exécution :**

```
declare -A my_colors_map=([red]="rouge" [blue]="bleu" [white]="blanc" [vert]="green" [black]="noir" )
```

### Obtenir la définition d'une fonction avec _typeset -pf_

Enfin, il est également possible d'afficher la définition d'une fonction avec
_typeset -pf_ :

**Code bash :**

```bash
#!/bin/bash
my_print() {
    local -r txt="$1"

    printf "%s\n" "$txt"
}
typeset -pf my_print
```

**Exécution :**

```
my_print ()
{
    local -r txt="";
    printf "%s\n" ""
}
```

## Les références en bash (_local -n_)

Tout comme il est possible de définir des références dans certains langages de
programmation (par exemple le C++), il est également possible de définir des
références en bash !

### Exemple C++

En C++, une référence se déclare avec le _&_.
Voici l'exemple d'une fonction qui incrémente de 1 la référence passée en
argument :

**Code C++ :**

```cpp
#include <iostream>

void my_inc(int& my_var)
{
    ++my_var;
}

int main()
{
    int my_int = 42;

    my_inc(my_int);
    std::cout << "my_int = " << my_int << std::endl;
}
```

**Exécution :**

```
my_int = 43
```

### Exemple bash

En _bash_, une référence se déclare avec l'option _-n_.
Voici le code équivalent au précédent programme C++ :

**Code bash :**

```bash
#!/bin/bash

my_inc() {
    local -n my_var="$1"

    ((my_var++))
}

main() {
    local -i my_int=42

    my_inc "my_int"
    printf "my_int = %i\n" $my_int
}

main
```

**Exécution :**

```
my_int = 43
```

## Exécuter une fonction bash à distance

Maintenant que les prérequis ont été présentés, voici comment exécuter une
fonction bash sur une machine distante.
Les exemples suivants vont du cas le plus simple (sans transmission d'arguments)
jusqu'à la récupération d'un tableau en retour de fonction.

### Hello world

Voici ci-dessous un programme simple "Hello world" qui exécute une fonction
bash dans un environnement bash distant (exécuté grâce à ssh).
La définition de la fonction _hello_world()_ est transmise dans l'instance
distante de bash avec _typeset -pf_ :

```bash
#!/bin/bash

# Définitions
hello_world() {
	printf "Hello world\n"
}

# Exécution du code distant
ssh exemple.org bash <<EOF
$(typeset -pf hello_world); hello_world
EOF
```

### Transmettre des arguments

Au lieu d'afficher un texte prédéfini, ce code bash propose maintenant
d'afficher les chaînes de caractères d'un tableau.
Ce tableau est donc transmis dans l'instance distante de bash avec _typeset -p_
:

```bash
#!/bin/bash

# Définitions
declare -ra my_words=("Hello " "world")

print_words() {
	local -n args="$1"

	for s in "${args[@]}"; do
		printf "%s" "$s"
	done
	printf "\n"
}

# Exécution du code distant
ssh exemple.org bash <<EOF
$(typeset -p my_words); $(typeset -pf print_words); print_words "my_words"
EOF
```

### Récupérer des résultats

Dans cet exemple, la fonction _get_words()_ exécuté sur une machine distante
doit retourner un tableau.
Pour que ce tableau soit récupérable, la fonction exécutée à distance retourne
du code bash évalué par l'instance principale grâce à _eval_ :


```bash
#!/bin/bash

# Définitions
get_words() {
	local array_name="$1"

	printf "${array_name}[0]=\"Hello \"\n"
	printf "${array_name}[1]=\"world\"\n"
}

# Exécution du code distant et stockage du résultat
declare -a my_array=()
eval $(ssh "exemple.org" bash <<EOF
$(typeset -pf get_words); get_words "my_array"
EOF
)

# Affichage du résultat
for s in "${my_array[@]}"; do
	printf "%s" "$s"
done
printf "\n"
```

## Exemple : récupération d'informations sur une machine distante

Le script ci-dessous permet de récupérer les informations d'une machine distante
dont le nom est donné en argument.
Pour cela, il exécute les commandes du tableau _INFO_COMMANDS_ et stocke leurs
résultats dans le tableau associatif _host_info_.

```bash
#!/bin/bash

set -o errexit -o errtrace -o pipefail -o nounset

declare -rA INFO_COMMANDS=(
	["debian_version"]="cat /etc/debian_version"
	["kernel_version"]="uname -r"
)

get_cmd_results() {
	local -rn commands="$1"
	local -r results="$2"

	for c in "${!commands[@]}"; do
		printf "${results}[$c]=$(${commands[$c]})\n"
	done
}

exec_remote() {
	local -r dest="$1"
	local -r code="$2"

	ssh $dest bash <<EOF
${code}
EOF
}

main() {
	local -r dest="$1"
	local -A host_info=()

	local -r code="\
$(typeset -p INFO_COMMANDS);\
$(typeset -pf get_cmd_results);\
get_cmd_results INFO_COMMANDS host_info"
	eval "$(exec_remote "$dest" "$code")"

	for info in "${!host_info[@]}"; do
		printf "${info}: ${host_info[$info]}\n"
	done
}

main "$@"
```

**Exécution :**

```
debian_version: 9.5
kernel_version: 4.9.0-8-amd64
```

## Ressources

* Manuel du programmeur de bash : <http://man7.org/linux/man-pages/man1/bash.1.html>
* Un Wiki regroupant une mine d'information sur bash : <http://wiki.bash-hackers.org>
