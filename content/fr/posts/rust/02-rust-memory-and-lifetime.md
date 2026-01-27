+++
date = '2026-01-18'
draft = false
title = 'Introduction à Rust (2) : gestion de la mémoire et lifetime'
tags = ['rust', 'learn', 'programming']
+++

Si vous avez besoin d'un rappel rapide sur comment un programme stocke de la
mémoire durant son exécution ou sur les différences entre la pile et le tas,
voici une [petite piqure de rappel](../../other/heap-vs-stack) pour vous rafraichir
la mémoire [^1].

## Pas de gargabe collector mais quand même du chocolat ?

Pour ne pas avoir de problème dé mémoire, de nombreux langages (C#, Java, Python,
Javascript, ...) ne laissent pas les développeurs choisir où ils allouent la mémoire[^2].
La plupart des objets sont stockés automatiquement sur le tas. Derrière, un mécanisme
appelé le *garbage collector* (ou ramasse miettes pour les amoureux de la langue
française) vérifie régulièrement si des objets alloués sur le tas sont encore
utilisés pour nettoyer l'espace mémoire qu'ils prennaient. L'avantage est la
simplicité d'utilisation; la contre partie, un impact sur les performances.
De plus, le *garbage collector* est géré automatiquement, on ne sait donc jamais
quand il va faire sa vérification.

D'autres languages comme C, C++ et Rust ne possèdent pas de **garbage collector**.
Par défaut, les objets sont alloués sur la pile (si c'est possible) et la mémoire
qu'ils prennaient supprimées dès que les variables sortent du *scope* de leur création.

```rust
fn main()
{
    let age = 18; // 18 est stocké sur la pile
    
    {
        let pi = 3.14; // 3.14 est stocké sur la pile
    } // le scope de pi finit, 3.14 est supprimée de la pile
} // 18 est supprimé de la pile
```

Gros avantage, le mécanisme est complétement déterministe et l'on tire tous les
avantages de performance de la pile. Problème, comment créer une variable donc on
ne connait pas la taille à la compilation ? Par exemple, une variable de type
`DatabaseService` où `DatabaseService` serait une interface (en Rust on parle de
`trait`). Rust s'est inspiré de C++11, pour gérer ces cas avec ses *pointeurs intelligents*
et son principe de [RAII](https://fr.wikipedia.org/wiki/Resource_acquisition_is_initialization).
Ces pointeurs stockés sur la pile permettent d'allouer de la mémoire sur le tas,
quand ils sortent de leur *scope* de création, ils nettoient automatiquement la mémoire
vers laquelle il avait une référence.

Le pointeur intelligent le plus simple est `Box<T>` en Rust :

```rust
fn main()
{
    let integer_stack = 12; // alloué sur la pile
    
    {
        // alloue 12 sur le tas, le pointeur sur la pile
        let integer_heap = Box::new(12); 
    } // le pointeur est supprimée de la pile et supprimée
    // en même temps automatiquement 12 sur le tas.
}
```

A la différence de C (via `malloc` et `free`) et C++ (via `new` et `delete`),
il n'y aucune méthode pour allouer et dé-allouer sur le tas en deux étapes,
aucun risque d'oublier de nettoyer et donc de faire fuite de la mémoire [^3].

[^1]: Ceci est un jeu de mot trop facile et pas forcément pertinent.
[^2]: C# donne un peu plus de liberté via les structs ou des mots clés tels que `stackalloc`.
[^3]: il est en réalité possible de faire des *memory leak* avec des références cycliques mais cela sort du cadre de l'article.
