+++
date = '2026-01-26'
draft = false
title = 'Introduction à Rust (2) : gestion de la mémoire et lifetime'
tags = ['rust', 'learn', 'programming']
+++

Si vous avez besoin d'un rappel rapide sur comment un programme stocke de la
mémoire durant son exécution ou sur les différences entre la pile et le tas,
voici une [petite piqûre de rappel](../../appendix/heap-vs-stack) pour vous rafraichir
la mémoire [^1].

## Pas de gargabe collector mais quand même du chocolat ?

Pour éviter les problème de mémoire, de nombreux langages (C#, Java, Python,
Javascript, ...) ne laissent pas les développeurs choisir où ils allouent la mémoire[^2].
La plupart des objets sont stockés automatiquement sur le tas. En arrière-plan,
un mécanisme appelé le *garbage collector* (ou ramasse miettes pour les amoureux
de la langue française) vérifie régulièrement si des objets alloués sur le tas sont
encore utilisés. Si ce n'est plus le cas, il nettoie l'espace mémoire qu'ils prennaient.
L'avantage est la simplicité d'utilisation. La contrepartie, un impact sur les performances.
De plus, le *garbage collector* est géré automatiquement et pas forcément
complétement déterministe, on ne peut pas prévoir quand il s'exécute.

D'autres langages comme C, C++ et Rust ne possèdent pas de **garbage collector**.
Par défaut, les objets sont alloués sur la pile (si c'est possible) et la mémoire
qu'ils occupaient est supprimée dès que les variables sortent de leur *scope*.

```rust
fn main()
{
    let age = 18; // 18 est stocké sur la pile
    
    {
        let pi = 3.14; // 3.14 est stocké sur la pile
    } // le scope de pi finit, 3.14 est supprimée de la pile
} // 18 est supprimé de la pile
```

Gros avantage, le mécanisme est déterministe et l'on tire tous les
avantages de performance de la pile. Problème: comment créer une variable dont
on ne connaît pas la taille à la compilation ? Par exemple, une variable de type
`DatabaseService` où `DatabaseService` serait une interface (en Rust on parle de
`trait`). Rust s'est inspiré de C++11, pour gérer ces cas avec ses *pointeurs intelligents*
et son principe de [RAII](https://fr.wikipedia.org/wiki/Resource_acquisition_is_initialization).
Ces pointeurs stockés sur la pile permettent d'allouer de la mémoire sur le tas,
quand ils sortent de leur *scope* de création, ils nettoient automatiquement la mémoire
vers laquelle ils avaient une référence.

Le pointeur intelligent le plus simple est l'object générique `Box<T>` en Rust :

```rust
fn main()
{
    let integer_stack = 12; // alloué sur la pile
    
    {
        // alloue 12 sur le tas, le pointeur sur la pile
        let integer_heap = Box::new(12); 
    } // le pointeur est supprimé de la pile et supprime
    // en même temps automatiquement 12 sur le tas.
}
```

A la différence de C ou de C++ sans les *smart pointer*, la dé-allocation de la mémoire
allouée sur la pile est automatique, il n'y a pas de risque d'oublier d'appeler
`free` ou `delete` et donc de créer des memory leaks[^3].

## Durée de vie ou comment supprimer les *dangling references*

### Un premier cas de base

*Note :* Dans cette section, pointeur et référence seront considérés comme équivalents.

*Rappel :* un [*dangling reference/pointer*](https://fr.wikipedia.org/wiki/Dangling_pointer)
est une référence vers un espace mémoire non valide. Dans les langages avec *garbage
collector*, les objets sont en vie tant qu'on a besoin d'eux, références comprises,
il n'y a donc pas ce problème.


Exemple en C tiré de la page wikipédia des *dangling pointer*:

```c
{
  char* dp = NULL;
  // ...
  {
    char c = 'J';
    dp = &c;
  } 
  // c n'est plus dans le scope, la mémoire a été nettoyée,
  // dp pointe donc vers une zone mémoire non valide
  dp = ...; // BOUMMMM 
}
```

Pour éviter ça, le compilateur vient encore à notre rescousse. A la compilation,
il vérifie la durée de vie des variables afin de valider ou d'invalider les références.
Ainsi, si l'on écrit :

```rust
let x;
    
    {
        let y = 'J';
        x = &y;
    }
    
    println!("{x}");
```

Le compilateur nous sort le message d'erreur le messasuivant : `y does not live long
enough`. En C ou C++, ce bug pourrait passer inaperçu jusqu’à l’exécution, alors
qu’en Rust il est détecté à la compilation.

### Le cas des retours de fonctions

Un autre cas courant, qui ne poserait pas de problème à un compilateur C ou C++[^4]:

```rust
// Ne faites pas attention au `a, on en parle bientot
fn get_number<'a>() -> &'a i32 { 
    let x = 12;
    
    return &x;
}
```

mais qui ne compile pas en Rust et retourne le message d'erreur :
`cannot return reference to local variable x` pour nous éviter
des problèmes.

### Les cas un peu complexe

Jusque là, le langage se débrouillait tout seul; cependant, parfois il faut l'aider.
Prenons ce code très inspiré du [tutorial officiel](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html):

```rust
fn longest_word(word1 : &String, word2: &String) -> &String 
{
    if word1.len() > word2.len(){
        return word1;
    } else {
        return word2;
    }
}
```

On souhaite, une fonction qui nous retourne le mot le plus long parmi deux entrées,
il n'y a pas de raison que la fonction prenne possession des *inputs*,
on travaille donc avec des références. S'il n'y avait qu'une référence en entrée,
Rust aurait attribué à la référence de sortie sa durée de vie (cf [*lifetime elision*](https://doc.rust-lang.org/nomicon/lifetime-elision.html)).
Mais ici, avec deux références en entrée, le langage ne peut rien deviner.
Quand le compilateur ne peut pas deviner, on utilise les *lifetime annotations*.
C'est une syntaxe qui permet de décrire des liens entre des durées de vie de
reférences. La syntaxe est la suivante:
elle commence par une apostrophe `'` puis et suivie d'un nom générique
(souvent `a, b, c, ect`).

```rust
fn longest_word<'a>(word1 : &'a String, word2: &'a String) -> &'a String 
{
    if word1.len() > word2.len(){
        return word1;
    } else {
        return word2;
    }
}
```

Ici on spécifie que la méthode est générique par rapport à la durée de vie nommée
`a` de la manière suivante : la sortie de la fonction aura la même durée de vie
que la durée de vie des deux références en entrée. La durée de vie de la référence
retournée doit être compatible avec celles des deux entrées; donc elle est bornée
par la plus courte.

```rust
let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1, string2);
        println!("The longest string is {result}");
    }
println!("The longest string is {result}"); // ne compile pas
```

Dans ce cas, c'est `string2` vit le moins longtemps donc `result` aura sa
durée de vie.

Rust a repris une forme de syntaxe que l'on retrouve dans de nombreux langages
pour dire qu'une méthode/object est générique par rapport à un type mais en l'adaptant
à une durée de vie pour garantir la sécurité des références. D'ailleurs, Rust a aussi
de la généricité sur les types, les deux syntaxes peuvent donc co-exister.

## Conclusion

Chapitre dense qui montre une grosse particularité du langage avec sa généricité
sur la durée de vie qui peut surprendre au début ! J'espère avoir su garder mon
explication concise mais claire ! En résumé, Rust combine *ownership*, *borrowing*
et *lifetimes* pour garantir la sécurité mémoire, et ce sans garbage collector.

On continue avec une autre spécificité, [les énumérations et la gestion des erreurs](03-rust-enum-and-error-handling)!

[^1]: Ceci est un jeu de mot trop facile et pas forcément pertinent.
[^2]: C# donne un peu plus de liberté via les structs ou des mots clés tels que `stackalloc`.
[^3]: il est en réalité possible de faire des fuite mémoires avec des références cycliques mais cela sort du cadre de l'article.
[^4]: je suis médisant, je viens de tester, j'ai heureusement un warning, mais ça reste léger.
