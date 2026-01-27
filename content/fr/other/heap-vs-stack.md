+++
date = '2026-01-19'
draft = false
title = 'Différence entre la pile et le tas'
+++

## Rappel

Avant d'entrer dans le vif du sujet, faisons un très rapide rappel des possibilités
pour un programme pour stocker des informations en mémoire durant son exécution.

Au *runtime*, un programme peut stocker les données dans deux endroits:

### La pile

La pile (ou *stack* en anglais) est endroit ou l'on peut stocker des variables
locales, à chaque entrée dans une fonction, un bloc de mémoire est crée pour stocker
ses données. Une fois que l'on sort du corps de la fonction, le bloc mémoire est
automatiquement nettoyé.

**Avantages :**

- accès mémoire un peu plus rapide que sur le tas
- la donnée est rangée de manière séquentielle, ce qui permet des optimisations CPU
- pas besoin d'allouer/dé-allouer de la mémoire à la main

**Inconvénients :**

- capacité de stockage limitée
- on ne peut y stocker que des objets dont la taille est connue (donc adios les listes
ou les chaines de caractères modifiables).

### Le tas

Le tas ou (*heap* en anglais) est une zone mémoire bien plus grande ou l'on peut
allouer/dé-allouer/ré-allouer de la mémoire. On peut donc stocker de *gros* objets
ou des objets dont la taille n'est pas fixe.

**Avantages :**

- Pas de contrainte sur ce que l'on stocke

**Inconvénients :**

- selon les langages (coucou C), il faut gérer l'allocation à la main, il y a donc
des risque de fuite mémoire.
- les objets ne sont pas stockés de manière contigus, le CPU ne peut donc pas itérer
sur eux de la manière la plus optimale.
