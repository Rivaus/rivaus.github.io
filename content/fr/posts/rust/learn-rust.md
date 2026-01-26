+++
date = '2026-01-16'
draft = false
title = 'Introduction à Rust'
tags = ['Rust', 'Learn', 'Programming']
+++

## Introduction

Je me suis intéressé à [Rust](https://rust-lang.org/fr/) en automne
2025 dans le but d'apprendre [un nouveau langage différent de C#](../why-learn-new-programming-langage).

Je l'ai choisi quand j'ai découvert que Rust était, encore cette année, le
*the most admired programming langage* [des sondages annuels StackOverflow](https://survey.stackoverflow.co/2025/technology).
Le langage est peut-être donc un peu surcoté mais doit posséder des caractéristiques
intéressantes !

## Les principales caractéristiques

Rust a une syntaxe bien différente de celle de C#,
ne propose pas de *garbage collector* et possède une mascotte : tout semblait coché
pour lui donner sa chance !

Si on cherche sur Internet les caractéristiques du langage, on tombe sur :

- sécurisé du point de vue de la mémoire et de la programmation concurrente : le
langage semble promettre des programmes moins buggés,
- *blazingly fast* : on peut donc s'attendre à un langage assez *bas niveau*
: le langage est d'ailleurs [entré récémment offciellement dans le kernel Linux](https://thenewstack.io/rust-goes-mainstream-in-the-linux-kernel/)
au côté de C, et Windows a également annoncé vouloir réécrire des parties de leur
OS avec Rust. Cependant, il semble aussi utilisé pour des applications
plus haut niveau comme des serveurs web,
- une courbe d'apprentissage plutôt élévée.

Concrètement, comment cela se transcrit-il ? Quelles sont ses spécificités ?

## Le voyage

J'ai décidé de découper chaque fonctionnalité que j'ai trouvé intéressante dans un
article dédié : c'est donc purement subjectif, absolument pas exhaustif
et cela a pour but de fournir une petite introduction au langage :

- [Introduction à Rust (1) : ownership & borrowing](01-rust-ownership-borrowing)
- [Introduction à Rust (2) : gestion de la mémoire et lifetime](02-rust-memory-and-lifetime)
- [Introduction à Rust (3) : les enums et gestion des erreurs]
- [Introduction à Rust (4) : orienté objet ?]
- [Introduction à Rust (5) : un ecosystème bien pensé]
- [Introduction à Rust (6) : programmation concurrente]
