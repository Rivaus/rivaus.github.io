+++
date = '2026-01-17'
draft = false
title = 'Introduction à Rust (1) : ownership & borrowing'
tags = ['Rust', 'Learn', 'Programming']
+++

Cet article s'inscrit dans [ma découverte de Rust](learn-rust).

Prenons un exemple simple dans un langage comme Python, Java ou C# (ici C#
est choisi):

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

Problème, cela ne compile pas et Rust retourne l'erreur `borrow of moved value a`.
En Rust, une variable possède un unique propriétaire, quand on appelle `modify_foo`,
on transfère la propriété (*ownership*) de Foo à la fonction : la variable `a`
n'est donc plus utilisable dans `main`. Heureusement, on peut partager une référence
d'un object sans en transférer la propriété. La référence ne fait que *borrow*
(emprunter en français) l'objet pour une durée déterminée (plus d'info sur cette
durée dans [l'article suivant](02-rust-memory-and-lifetime)) comme ceci :
`let reference_to_a = &a`. On peut aussi utiliser `&` pour signifier qu'une
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
définir les variables comme modifiables avec le mot-clé `mut`.
Idem pour les références, par défaut, la référence pointera vers une valeur immuable
(même si va valeur est modifiable), iimmaublel faut donc changer également la signature
de notre fonction:

```rust
fn main() {
    let mut a = Foo::new(); // a devient modifiable
    modify_foo(&mut a); // On crée une référence vers une valeur modifiable
    println!("{a:?}");
}

fn modify_foo(foo : &mut Foo) { // la méthode requiert une référence modifiable
    foo.property = 16;
}
```

Le code fonctionne et affiche `Foo { property: 16 }`, YOUPI !

## Reègles du borrowing : on peut emprunter mais pas n'importe comment

Modifions un peu notre code :

```rust
fn main()
{
    let mut a = Foo::new();
    let b = &mut a; // Soyons fous, créons une seconde référence pour modifier a
    modify_foo(&mut a);
    b.property = 20;
}

fn modify_foo(foo : &mut Foo) {
    foo.property = 16;
}
```

Le compilateur nous bloque à nouveau : `cannot borrow a as mutable more than
once at a time`. Il nous previent d'une règle fondamentale du *borrowing*, on
peut créer :

- soit N références actives vers un objet immuable
- soit une et une seule référence active vers un object modifiable

Cela est important pour prévenir les *data race* : ici il n'est pas sécurisé de lire
ou de modifier a dans `modify_foo` via une référence car une autre référence
modifiable `b` a été créée et pourrait modifier `a` a tout moment
(dans un autre *thread* par exemple) ! Le compilateur est exigeant, mais pour notre
bien, pour évider d'éventuels bugs.

*Note: si on commente la ligne `b.property = 20;`, le compilateur est assez intelligent
pour voir que le caractère modifiable de la référence `b` est inutile et ainsi le
code compile*

On peut régler le problème de plusieurs manières :

- Ici on modfie `a` via `b` avant de créer une nouvelle référence modifiable vers
`a`. Puisque la variable n'est pas utilisée après, le compilateur comprend qu'il
n'y a aucun risque :référence

```rust
fn main()
{
    let mut a = Foo::new();
    let b = &mut a;
    b.property = 20;
    modify_foo(&mut a); // C'est bon, b ne sera plus utilisée après
}

fn modify_foo(foo : &mut Foo) {
    foo.property = 16;
}
```

- On peut directement passer `b` en référence a `modify_content` : on
n'a qu'une seule référence modifiable vers a, donc aucun risque de *data race*.

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

## Conclusion

Voici la fin de nos premiers pas dans le monde de Rust avec ce concept fondamental
de l'*ownership*. Il peut paraitre contraignant aux premiers abords mais vise à
réduire les sources de bug et permet au compilateur des optimisations impossibles
sans ces informations ! Remarquez aussi que le compilateur a su nous donner
de bons conseils à chaque fois pour régler nos problèmes ! De plus, toutes
ces vérifications se font à la compilation et ne viennent donc pas impacter les
performances de nos applications !

Autre élément que nous avons découvert ici : tout est explicite en Rust,
la lecture du code est d'autant plus facilité. Les notions de passer un argument
par *propriété*, référence ou référence modifiable fonctionne également quand on
définit des méthodes d'instance d'un objet :

```rust
struct Foo {
    property : u32
}

impl Foo {
    // self renvoie à l'instance, comme en Python, équivalent de this en C# ou Java
    
    // Print ne modfie pas l'objet, donc une référence suffit
    fn print_data(&self) {
        println!("Foo's property equals {}", self.property)
    }
    
    // On voit directement que cette méthode modifie l'objet
    fn set_property(&mut self, new_value:u32) {
        self.property = new_value;
    }
    
    // Prend "soi-même possession de soi", impossibles
    // d'utiliser l'object après
    fn consume(self) {
    }
}
```

C'est un élément que j'apprécie beaucoup, C++ propose la même chose
mais avec un choix inverse : la modication possible par défaut, l'immuabilité
en exception, je préfère le choix de Rust.

J'espère que vous avez tenu le coup, rendez vous dans le
prochain épisode pour mieux comprendre ces choix :
[la gestion de la mémoire](02-rust-memory-and-lifetime) !
