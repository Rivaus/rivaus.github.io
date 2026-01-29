+++
date = '2026-01-19'
draft = false
title = 'Différence entre la pile et le tas'
+++

## Rappel

Voici un très rapide rappel des possibilités pour un programme pour stocker des
informations en mémoire durant son exécution.

Au *runtime*, un programme peut stocker les données dans deux emplacements:

### La pile

La pile (ou *stack* en anglais) est endroit où l'on peut stocker des variables
locales. A chaque entrée dans une fonction, un bloc de mémoire est créé pour stocker
ses données (variables, contexte, adresses de retour, ...). Une fois que l'on
sort du corps de la fonction, le bloc mémoire est automatiquement nettoyé.

**Avantages :**

- accès mémoire un peu plus rapide que sur le tas
- la donnée est rangée de manière séquentielle, ce qui permet des optimisations CPU
- pas besoin d'allouer/dé-allouer de la mémoire à la main

**Inconvénients :**

- capacité de stockage limité (dépend du système/language)
- on ne peut y stocker que des objets dont la taille est connue. Exemple d'entitées
dont la taille n'est pas connue :
  - les listes
  - les chaînes de caractères modifiables
  - les variables dont le type est une interface, on ne connait pas l'implémentation
derrière, donc on ne peut pas prévoir de taille de mémoire

### Le tas

Le tas ou (*heap* en anglais) est une zone mémoire bien plus grande où l'on peut
allouer/dé-allouer/ré-allouer de la mémoire. On peut donc stocker de *gros* objets
ou des objets dont la taille n'est pas fixe.

**Avantages :**

- Pas de contrainte sur ce que l'on stocke

**Inconvénients :**

- pour les langagues dont la mémoire n'est pas gérée automatiquement (coucou C ),
il faut gérer l'allocation à la main, il y a donc des risque de fuite mémoire.
- les objets ne sont pas stockés de manière contigus, le CPU ne peut donc pas itérer
sur eux de la manière la plus optimale.

## Exemple

Exemple en C :


```c
// Sur la pile
int x = 5;

// Sur le tas, il ne faudra donc pas oublier de libérer la mémoire
int* y = malloc(sizeof(int));
```
