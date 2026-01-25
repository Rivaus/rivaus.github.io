+++
date = '2026-01-16'
draft = false
title = 'Introduction à Rust'
tags = ['Rust', 'Learn', 'Programming']
+++

## Introduction

J'ai commencé à apprendre Rust en octobre/novembre 2025 dans le but d'apprendre
[un nouveau langage différent de C#](../why-learn-new-programming-language).

Je l'ai choisi quand j'ai découvert que Rust était encore cette année le
*the most admired programming language* [des sondages annuels StackOverflow](https://survey.stackoverflow.co/2025/technology),
le language est peut-etre donc un peu *over hypé* mais doit posséder des caractéristiques
intéressantes !

## Les principales caractéristiques

Rust a un syntax bien différente de C# (proche du Python), ne propose pas de
*garbage collector* et possède une mascotte : tout semblait coché pour lui
donner une chance !

Si on cherche sur Internet les caractéristiques du langage, on tombe sur :

- sécurisé d'un point de vue mémoire ou programmation concurrante : le language
semble promettre des programmes moins buggées
- *blazingly fast* : on peut donc s'attendre à un language assez *bas niveau*
: le language est d'ailleurs [entré récémment offciellement dans le kernel Linux](https://thenewstack.io/rust-goes-mainstream-in-the-linux-kernel/)
au côté de C et Windows a également annoncé vouloir réécrire des parties de leur
OS avec aux côtés de C). Cependant, il sembel aussi utilisé pour des applications
plus au niveau comme des servers webs.
- une courbe d'apprentissage plutôt élévée car le langage aurait des concepts
assez stricts

Concrétement comment cela se transcrit-il ? Quels fonctionnalités sont éloigées du
fonctionnement de C# ?

## Le voyage

J'ai décidé de découper chaque fonctionnalité que j'ai trouvé importante dans un
article dédié : c'est donc purement subjectif, absolument pas exhaustif et à pour
but de plutôt fournir une petite introduction au langage :

- [Introduction à Rust (1) : ownership & borrowing](/rust/ownership-borrowing)
