+++
date = '2026-01-17'
draft = false
title = 'Introduction à Rust (1) : ownership & borrowing'
tags = ['Rust', 'Learn', 'Programming']
+++

Prenons un exemple simple en C#:

```csharp
var a = new Foo();
a.Property = 12;
var b = a;
Console.WriteLine(a.ToString());
```

En Rust :

```rust
let a = Foo::new();
a.property = 12;
let b = a;
println!("{a:?}");
```

Problème, ça ne compile pas `a is not declared as mutable`. En effet, par défaut,
les variables sont immuables, il faut definir la variable comme muable :

```rust
let mut a = Foo::new();
a.property = 12;
let b = a;
println!("{a:?}");
```

Le compilateur n'est toujours pas content avec la dernière ligne : `borrow of
moved value: a`. En effet, en Rust, une variable possède un unique propriétaire,
lors de la ligne `let b = a`, la propriété de *Foo* est transférée à la
variable b, il n'est donc plus valide d'utiliser a.
