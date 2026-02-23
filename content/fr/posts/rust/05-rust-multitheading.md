+++
date = '2026-02-22'
draft = false
title = 'Introduction à Rust (5) : du multithreading sans souci ?'
tags = ['rust', 'learn', 'programming']
+++

Dans cet article, je vais tenter de vous expliquer comment [les spécificités](learn-rust.md)
du langage Rust rendent l'écriture de code parallélisable plus sûr.
Quand on parle d'exécution de code parallèle (ou *multithreadé* ),
on évoque la programmation concurrente. Rust possède également le support de l'asynchrone
avec le duo [async/await](https://doc.rust-lang.org/book/ch17-00-async-await.html).
Cependant, je trouve son utilisation relativement similaire aux autres langages,
donc je ne l'évoquerai pas ici [^1].

## Analogie entre C# et Rust

Prenons un exemple classique pour illustrer les problèmes que peut poser le code
parallèle en C#.

```c
public class Counter {
    public int n;

    public void Increment() {
        this.n++;
    }
}

static void Main()
{
    var counter = new Counter();
    Thread t1 = new Thread(() => IncrementCounter(counter));
    Thread t2 = new Thread(() => IncrementCounter(counter));

    t1.Start();
    t2.Start();

    t1.Join();
    t2.Join();

    Console.WriteLine($"Valeur finale du compteur : {counter.n}");
}

static void IncrementCounter(Counter counter)
{
    for (int i = 0; i < 1000000; i++)
    {
        counter.Increment();
    }
}
```

Ici le résultat final peut varier et ne pas toujours valoir 200000.
En effet l'opération `this.n++;` n'est pas [atomique](https://fr.wikipedia.org/wiki/Atomicit%C3%A9_(informatique)):
des incrémentations peuvent se perdre entre les deux *threads*.
Cet exemple très simple peut faire sourire le développeur chevronné que vous
êtes : vous auriez directement anticipé le problème et utilisé le mot clé
`lock` pour ne pas avoir de problème !

```csharp
//...
for (int i = 0; i < 1000000; i++)
{
  lock(counter) {
    counter.Increment();
  }
}
//...
```

Cependant :

- dans de vrais projets, le contexte d'exécution n'est pas aussi évident et vous
pourriez ne pas voir que votre code s'exécute dans deux *threads* différents,
- même sur un exemple simple, C# nous a laissé faire *tranquillou bilou*.

Essayons de faire la même chose en Rust :

```rust
use std::thread;

struct Counter {
    n: u32,
}

impl Counter {
    fn increment(&mut self) {
        self.n += 1;
    }
}

fn main() {
    let mut counter = Counter { n: 0 };

    let t1 = thread::spawn(|| {
        for _ in 0..100000 {
            counter.increment();
        }
    });

    let t2 = thread::spawn(|| {
        for _ in 0..100000 {
            counter.increment();
        }
    });

    t1.join().unwrap();
    t2.join().unwrap();

    println!("Valeur finale {}", counter.n);
}
```

Première erreur de compilation : `closure may outlive the current function,
but it borrows counter, which is owned by the current function`.
En effet, par défaut, la fonction en argument de `thread::spawn`
récupère notre compteur par référence. Or, il n’y a aucune garantie que la
variable survive aussi longtemps que le thread créé :
le compilateur nous bloque.

Il nous suggère de transférer la propriété du compteur dans le *thread* créé
avec le mot clé `move`:

```rust
//...
let t1 = thread::spawn(move || {
    for _ in 0..100000 {
        counter.increment();
    }
});

let t2 = thread::spawn(move || {
    for _ in 0..100000 {
        counter.increment();
    }
});
//...
```

Mais c'est alors le drame, car un object ne peut avoir qu'un
[propriétaire](01-rust-ownership-borrowing.md). Le compilateur
nous bloque à nouveau.

## Comment résoudre le problème ?

Dans l'article sur la [mémoire et la durée de vie], j'évoquais la notion
de pointeur intelligent avec son implémentation la plus simple `Box<T>`.
Il faut savoir qu'il existe un autre pointeur de base `Rc<T>`
(pour `Reference Counting`) qui permet d'avoir plusieurs références vers un objet.
Néanmoins, si nous tentons de l'utiliser ici, nous sommes aussi bloqués
car le type `Rc<T>` n'est pas autorisé à être transféré entre deux threads [^2].

Heureusement, il existe une version de ce pointeur autorisée à faire cela:
`Arc<T>` pour *Atomic Reference Counting*. Mais la vie serait trop facile si tous
nos problèmes étaient réglés. Le souci avec `Rc<T>` et `Arc<T>` c'est qu'on
ne peut pas modifier la valeur à l'intérieur. Dans notre cas, nous sommes obligé
de rajouter une encapsulation avec la structure de donnée `Mutex<T>` qui elle
autorise la modification de manière *thread-safe*. Puisque du code parle
plus que de longs mots:

```rust
use std::{
    sync::{Arc, Mutex},
    thread,
};

struct Counter {
    n: u32,
}

impl Counter {
    fn increment(&mut self) {
        self.n += 1;
    }
}

fn main() {
    let counter = Arc::new(Mutex::new(Counter { n: 0 }));
    
    let counter_01 = counter.clone(); // On crée une référence pour le thread 1
    let t1 = thread::spawn(move || {
        for _ in 0..100000 {
            let mut c = counter_01.lock().unwrap(); // ---+ lock counter
            c.increment();                          //    + dans le scope
        }                                           // ---+ du mutex
    });

    let counter_02 = counter.clone(); // On crée une référence pour le thread 2
    let t2 = thread::spawn(move || {
        for _ in 0..100000 {
            let mut c = counter_02.lock().unwrap(); // ---+ lock counter
            c.increment();                          //    + dans le scope
        }                                           // ---+ du mutex
    });

    t1.join().unwrap();
    t2.join().unwrap();

    println!("Valeur finale {}", counter.lock().unwrap().n);
}
```

Et voilà ! Ca compile et nous sommes sûrs de toujours obtenir
200000 en résultat final !

Bonus :

- Sachez également que pour des objets aussi simples que des entiers, il
existe des structures de données telles que `AtomicU32` faites pour lire et
modifier des nombres sans risque dans du code parallèle.
- Les règles de Rust forcent parfois à envisager l'écriture de code parallèle
selon des motifs différents qui évitent les problèmes d'accès concurrent.
Par exemple, la documentation officielle cite celle du langage Go avec ce très
beau dicton : [Do not communicate by sharing memory; instead, share memory by communicating](https://go.dev/doc/effective_go#concurrency).
Pour son utilisation en Rust, encore une fois je vous recommande le [tutoriel officiel](https://doc.rust-lang.org/book/ch16-02-message-passing.html)!

## Conclusion

Finalement, la solution Rust est relativement similaire à celle de C#,
il doit utiliser un verrou quand on modifie l'objet pour être sûr
d'être le seul à le modifer. La syntaxe est plus verbeuse et plus
explicite en Rust mais en contrepartie nous avons été obligés de développer
une solution sans bug [^3], est-ce donc si grave d'écrire un peu plus de code ?
On notera également que Rust a vérifié la validité du code *multithreadé*
à la compilation, ce qui est un énorme avantage !

On arrête pour le contenu technique de cette série ! Dans la prochaine partie,
je vous introduirai l'écosystème du langage(**en cours d'écriture**).

[^1]: le code asynchrone Rust est similaire à celui de C#, Python,
Typescript, etc. Néanmoins certains cas d'utilisation spécifiques peuvent
nécessiter une compréhension assez fine -que je ne possède pas totalement-.
Si vous êtes curieux et que vous avez un peu approfondi le language de votre
côté, je vous suggère [cette vidéo](https://www.youtube.com/watch?v=9RsgFFp67eo).
[^2]: il me semblait trop long d'expliquer ici pourquoi, pour des débuts d'explication
pour satisfaire votre curiosité, je vous invite à lire [ceci](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html).
[^3]: dans ces introductions, je présente le langage comme étant assez magique
et résolvant tous les problèmes de la Terre, je tempérerai mes affirmations
dans la dernière partie de cette série !
