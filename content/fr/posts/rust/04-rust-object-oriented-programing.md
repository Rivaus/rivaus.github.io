+++
date = '2026-02-08'
draft = false
title = 'Introduction à Rust (4) : le langage est-il orienté objet ?'
tags = ['rust', 'learn', 'programming']
+++

Rust est-il orienté objet ? La question est pertinente car ce paradigme
est adopté par bon nombre de langages de programmation populaires. C'est
également le paradigme que je connais le plus.

Pour tenter d'y répondre, je vous propose d'étudier les principales
caractéristiques de la [programmation orientée objet](https://en.wikipedia.org/wiki/Object-oriented_programming)
une par une.

## La notion d'objet et d'encapsulation

Par *objet*, j'entends des structures qui regroupent des données et des
méthodes. Ces données et méthodes peuvent être plus ou moins accessibles
depuis l'extérieur des structures avec la notion de visibilité (donnée public ou
privée par exemple). Cela permet de supporter le concept d'[encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)).
Dans la plupart des langages orientés objet, ces structures sont définies par des
**classes** que l'on peut **instancier** pour obtenir des **objets**.

En Rust, nous n'avons pas de classe mais des **structs**.Le concept reste
cependant assez similaire: ces structures peuvent contenir de la donnée et des méthodes.

```rust
struct Library {
    users: Vec<String>,
    available_books : Vec<String>,
    borrowed_books: Vec<String>,
    budget:f32
}

impl Library {

    /// On définit un constructeur
    fn new(available_books: Vec<String>, budget: f32) -> Library {
        Library {
            available_books,
            budget,
            borrowed_books: Vec::new(),
            users: Vec::new()
        }
    }
    
    /// self signifie que c'est une méthode d'instance
    /// &self signifie que la méthode ne modifie pas l'instance
    pub fn available_books(&self) -> &Vec<String> {
        // Une méthode sans ; final équivant à faire un retour de valeur
        &self.available_books
    }
    
    /// &mut self indique que la méthode modifie l'objet
    pub fn borrow_book(&mut self, user: String, book: String)
    -> Result<String, String> {
        if !self.users.contains(&user){
            return Err("User not registered".to_string());
        }
        
        if let Some(book_index) = self.available_books
                .iter()
                .position(|b| *b == book) {
                
            self.available_books.remove(book_index);
            self.borrowed_books.push(book.clone());
            
            return Ok(book);
        } else {
            return Err("Book not available".to_string());
        }
    }
}

fn main()
{
    let mut library = Library::new(vec![
            "Sherlock Holmes".to_string(),
            "Martine à la mer".to_string()
        ],
        10000.0);
        
    match library.borrow_book("non-existing-user".to_string(),
        "Sherlock Holmes".to_string()){
      Ok(_) =>   println!("Borrow succeeded !"),
      Err(e) => println!("Impossible to borrow : {e}")
    };
    
    println!("Available books :");
    for b in library.available_books() {
        println!("- {b}");
    }
}
```

Petites explications :

- avec le mot clé `struct` on définit une structure en décrivant
les données qu'elle contient,
- avec `impl` on définit les méthodes liées à cette structure. Si les méthodes
prennent comme premier argument `self, &self, ou &mut self`, c'est que ce sont
des méthodes d'instances. Sinon, ce sont des méthodes dites associées qui s'appellent
directement depuis la nom de la structure. Par exemple, en Rust, les structures n'ont
pas de constructeur par défaut mais par convention, on définit souvent une méthode
associée nommée `new` pour construire l'objet,
- par défaut, les propriétés et méthodes ne sont accessibles que dans le module
où est définie la structure (généralement, le fichier de définition).
Avec le modificateur `pub`, les méthodes `available_books` et `borrow_book`
deviennent publiques et sont donc accessibles depuis l'extérieur du module,
- dans notre exemple, la méthode `borrow_book` retournera une erreur car
aucune méthode pour enregistrer un utilisateur n'a été définie.

## Héritage et polymorphisme

Rust ne possède pas de mécanisme d'[héritage](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)).
Certains en concluent donc que le langage ne propose pas de paradigme
orienté objet au sens strict. Rust fait le choix de suivre les critiques faites
à l'héritage [^1] pour lui préférer la [composition](https://en.wikipedia.org/wiki/Object_composition)
[^2] (c'est d'ailleurs quelque chose que nous pouvons appliquer dans des langages
*très objet* comme C# ou Java).

Le langage propose cependant la programmation par contrat via des interfaces qui
sont appelés **trait**.

```rust
// On partage un comportement via un trait
trait UserRepository {
    fn fetch_users(&self) -> Vec<String>;
}

struct SqliteDatabase;

impl UserRepository for SqliteDatabase{
    fn fetch_users(&self) -> Vec<String> {
        Vec::new()
    }
}

struct PgDatabase;

impl UserRepository for PgDatabase{
    fn fetch_users(&self) -> Vec<String> {
        Vec::new()
    }
}

fn main(){
    let sqlite_repos = SqliteDatabase;
    let pg_repos = PgDatabase;
    
    get_users(sqlite_repos);
    get_users(pg_repos);
}

fn get_users(repos: impl UserRepository) {
    let _ = repos.fetch_users();
}
```

Cela permet du [polymorphisme](https://en.wikipedia.org/wiki/Polymorphism_(computer_science)):
la fonction `get_users` fonctionne avec n'importe quel objet qui implémente
`UserRepository`. On dit que la fonction est *trait bound*.

Pour les petits curieux, sachez que dans ce cas, le compilateur va dupliquer la
définition de cette méthode pour chaque type d'implémentation qui lui sera passée.
On parle de *static dispatch*. Lorsqu’il est impossible de déterminer à la
compilation quel type concret sera passé en argument, le langage utilise le
[*dynamic dispatch*](https://doc.rust-lang.org/book/ch18-02-trait-objects.html).

## Conclusion

Nous avons vu - rapidement et sans entrer dans les détails - que Rust n'est
pas un langage orienté objet au sens strict. Cependant, il propose des concepts
similaires avec les `structs` ou les `traits` auquel il ajoute ses spécificités :
unique propriétaire, sécurité mémoire, etc.
Si vous souhaitez plus de détails sur ce sujet, sachez que le tutorial officiel
dédie un [chapitre](https://doc.rust-lang.org/book/ch18-00-oop.html)
complet sur la question !

Dans le prochain chapitre, nous verrons comment les spécificités de Rust
nous permettent d'écrire du code concurrant plus sécurisé ! (*en cours d'écriture*)

[^1]: [Why extends is evil](https://web.archive.org/web/20190224073940/https://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)
[^2]: [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)
