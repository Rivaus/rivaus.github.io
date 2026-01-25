+++
date = '2026-01-17'
draft = false
title = 'Introduction à Rust (1) : ownership & borrowing'
tags = ['Rust', 'Learn', 'Programming']
+++

Cet article s'inscrit dans [ma découverte de Rust](learn-rust).

Prenons un exemple simple dans un langage tel que Python, Java ou C# (ici choisi):

```cs
public static void Main()
{
  var a = new Foo();
  ModifyFoo(a);
  Console.WriteLine(a);
}

private static void ModifyFoo(Foo foo) {
  foo.Property = 16;
}
```

## Ownership & Borrowing : la loi du propriétaire unique

Tentons de faire pareil en Rust :

```rust
fn main() {
    let a = Foo::new();
    modify_foo(a);
    println!("{a:?}"); // :? est une syntaxe pour afficher le contenu d'un objet
}

fn modify_foo(foo : Foo) {
    foo.property = 16;
}
```

Problème, ça ne compile pas : `borrow of moved value a`.  En effet, en Rust,
une variable possède un unique propriétaire, quand on appelle `modify_foo`,
on transfère l'*ownership* de Foo à la fonction, la variable `a` n'est donc
plus utilisable dans `main`. Heureusement, on peut partager une référence d'un object
sans en transférer la propriété, la référence ne fait que *borrow* (emprunter en
français) l'objet pour une durée déterminée (plus d'info sur cette durée dans
[l'article suivant](02-rust-memory-and-lifetime)) comme ceci :
`let reference_to_a = &a`. On peut aussi utiliser `&` pour signfier qu'une
fonction ne capture qu'une référence vers un objet sans en prendre la possession:

```rust
fn main() {
    let a = Foo::new();
    modify_foo(&a); // la fonction emprunte la valeur juste pour son exécution
    println!("{a:?}");
}

fn modify_foo(foo : &Foo) { // Une référence me suffit, pas besoin de plus !
    foo.property = 16;
}
```

## L'immuabilité comme comportement pas défaut, l'explicite comme crédo

Le compilateur n'est toujours pas satisfait, il ne peut pas changer la valeur de
`foo.property` : par défaut, en Rust, les variables sont immuables, il faut explicitement
définir les variables comme muables avec le mot-clé `mut`.
Idem pour les références, par défaut, la référence pointera vers une valeur immuable
(même si va valeur est muable), il faut changer également la signature de notre fonction:

```rust
fn main() {
    let mut a = Foo::new(); // a peut être modifiée
    modify_foo(&mut a); // On crée une référence vers une valeur modifiable
    println!("{a:?}");
}

fn modify_foo(foo : &mut Foo) { // la méthode requiert une référence modifiable
    foo.property = 16;
}
```

Le code fonctionne et affiche `Foo { property: 16 }`, YOUPI !

Notes pour les gourmands :

- on ne peut créer une référence vers une valeur modiable que si la valeur a été
définie comme modifiable :

```rust
let a = Foo::new();
let ref_to_a = &mut a; // ne compile pas car a n'est pas mut.
```

- une référence peut être muable, c'est à dire qu'on peut changer vers quelle valeur
elle pointe, c'est indépendant du caractère muable ou non de la valeur pointée.

```rust
let a = Foo::new();
let b = Foo::new();
let mut ref_to_a = &a;
// on peut changer vers quoi la référence point mais pas les objets pointés
ref_to_a = &b; 
```

## Reègles du borrowing : on peut emprunter mais pas n'importe comment !

Modifions un peu notre code :

```rust
fn main()
{
    let mut a = Foo::new();
    let b = &mut a;
    modify_foo(&mut a);
    b.property = 20;
}

fn modify_foo(foo : &mut Foo) {
    foo.property = 16;
}
```

Le compilateur nous blqoque à nouveau : `cannot borrow a as mutable more than
once at a time`. Il nous previent d'une règle fondamentale du *borrowing*, on
peut créer des références :

- soit N références actives immuables vers un objet
- soit une et une seule référence muable active vers un object

Cela est important pour prévenir les *data race* : ici il n'est pas sécurisé de lire
ou modifier a dans `modify_foo` en créant une référence (muable ou non) car une
référence muable `b` a été créée et pourrait modifier `a` a tout moment !
Le compilateur est exigeant, mais pour notre bien, pour évider d'éventuels bugs.

*Note: si on commente la ligne `b.property = 20;`, le compilateur est assez intelligent
pour voir que le caractère muable de laréférence `b`est inutile et ainsi le code
compile*

On peut régler le problème de plusieurs manières :

- Ici on modfie `a` via `b` avant de créer une nouvelle référence muable vers `a`.
Puisque la variable n'est pas utilisée, le compilateur comprend qu'il n'y a aucun
risque :

```rust
fn main()
{
    let mut a = Foo::new();
    let b = &mut a;
    b.property = 20;
    modify_foo(&mut a);
}

fn modify_foo(foo : &mut Foo) {
    foo.property = 16;
}
```

- On peut directement passer `b` en référence a `modify_content` : on
n'a qu'une seule référence muable vers a, donc aucun risque de data race.

```rust
fn main()
{
    let mut a = Foo::new();
    let b = &mut a;
    b.property = 20;
    modify_foo(b);
}

fn modify_foo(foo : &mut Foo) {
    foo.property = 16;
}
```
