+++
date = '2026-01-29'
draft = false
title = 'Introduction à Rust (3) : enum & gestion des erreurs'
tags = ['rust', 'learn', 'programming']
+++

Les énumerations sont présentes dans de nombreux langages de programmation
avec une syntaxe souvent similaire à cela :

```cpp
enum Level {
    Low, Medium, High
}
```

Les énumérations définissent un type qui ne peut ne peut prendre qu'un nombre fini
de valeurs, rien de plus. Ici, une variable du type `Level` ne pourra valoir que
`Low`,`Medium` ou `High`. Rien de bien nouveau si vous êtes développeur.

En Rust, les énumérations sont un peu plus gourmandes, chaque valeur possible de
l'énumération peut avoir zéro, un ou plusieurs attributs que l'on peut lire :

```rust
enum Document {
    Book {
        title: String,
        author: String

    },
    Article(String), // un paramètre non nommé
}

fn print_document_info(doc: &Document) {
    match doc {
        Document::Book { title, author } => {
            println!("Livre : '{}' écrit par {}", title, author);
        }
        Document::Article(content) => {
            println!("Article : {}", content);
        }
    };
}
```

## Les énumérations pour définir une valeur optionnelle

La plupart des langages de programmation permettent de définir qu'une variable
puisse avoir une valeur ou non. Cela passe souvent par des objets spéciaux génériques:
[std::optional](https://en.cppreference.com/w/cpp/utility/optional.html) en C++ ou
[Nullable\<T>](https://learn.microsoft.com/fr-fr/dotnet/api/system.nullable-1?view=net-10.0).

En Rust, il existe une *enum* générique dans la librairie standard :

```rust
enum Option<T> {
    None,
    Some(T),
}
```

La capacité des énumérations à contenir des valeurs se marient bien avec
la manière de les traiter avec le mot clé `match` :

```rust
fn print_user_info(name: Option<String>)
{
    match name {
        Some(n) => println!("[User. Name {n}]"),
        None => println!("[User. Unknown]") // obliger de gérer ce cas
    };
}
```

`match` requiert que tous les possibilités soient traitées pour compiler, le langage
nous *force* donc à traiter le cas où le prénom ne serait pas renseigné. Cela évite
les bugs. Idem avec les syntaxes plus courtes pour traiter un cas spécifique, nous
sommes obligés[^1] de tester que nous avons bien une valeur :

```rust
let x: Option<String> = ...;
if let Some(n) = x {
    println!("[User. Name {n}]")
}
```

Cela renforce encore la sécurité de nos programmes.

## Les énumérations pour gérer des erreurs

Rust ne possède pas d'exception pour gérer les erreurs, mais deux autres mécanismes.
Le premier est *la panique* qui est à utiliser quand le programme est dans un état
*irrécupérable*, pour finir son exécution avec un message d'erreur. Le second
est moins brutal et témoigne du non-succès d'une opération (par exemple le *parsing*
d'une chaîne de caractères) et repose sur l'*enum* générique `Result<T, E>`:

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}

fn parse_to_int(entry:String) -> Result<i32, String>{...}

fn main(){
    match parse_to_int(String::from("12")) {
        Ok(nb) => println!("Number {nb}"),
        Err(e) => println!("Impossible to parse {e}")
    };
}
```

La méthode `parse_to_int` nous renvoie un résultat sous la forme d'un entier
si c'est un succès et d'une chaîne de caractères si c'est une erreur (un
message contenant plus d'information par exemple)[^2]. Rust insiste très fortement
pour que l'on traite le cas d'erreur puisque nous devons utiliser le *pattern match*
pour lire le valeur ou au moins vérifier que le résultat est de type `Ok`.
De plus, nous pouvons voir directement dans la signature de la fonction que
celle-ci peut échouer.

### Conclusion

Nous avons vu ici la spécificité des énumérations en Rust : elles peuvent contenir
des informations qui peuvent différer selon la valeur. Ces énumérations font parties
intégrantes de la gestion des erreurs et des valeurs optionnelles. Enfin, la manière
de récupérer les informations au sein des énumérations renforcent la sécurité des
programmes que l'on écrit : on doit vérifier qu'une variable optionnelle contienne
quelque chose, ou qu'une méthode susceptible d'échouer ait réussi avant de poursuivre.

Dans le prochain chapitre, nous répondrons à la question suivante : [Rust est-il
orienté objet](04-rust-object-oriented-programing) ? *(en cours d'écriture)*

[^1]: En réalité, `Option<T>` possède une méthode `unwrap()` qui permet de récupér directement
la valeur à l'intérieur si la variable vaut `Some(...)`. Cependant, cette méthode fera planter
votre programme si la variable vaut `None`, elle est donc réservée à du prototypage ou d'extrêmes
rares occasions.
[^2]: Dans un code plus idiomatique, on renvoit généralement plutôt un objet qui implémente
le *trait* [Error](https://doc.rust-lang.org/std/error/trait.Error.html) qu'une simple chaine
de caractères.
