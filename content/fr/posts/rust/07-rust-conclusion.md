+++
date = '2026-04-24'
draft = false
title = 'Introduction à Rust (7) : conclusion de cette aventure'
tags = ['rust', 'learn', 'programming']
+++

## Un langage attrayant

Après 7 petits articles, il est temps de finir ce voyage !
J'ai beaucoup aimé découvrir Rust, je comprends maintenant la popularité du langage.
Le langage est également un très bon outil complémentaire à d'autres langages
tels que C#, Java ou Python : son absence de garbage collector, son aspect multi-
paradigme, ses spécificités (*borrowing*, *lifetime*, enum avec champs, etc...)
demandent une manière de penser différente qui nous apprend beaucoup !
De plus, ses mécanismes de sécurité permettent des implémentations relativement
bas niveau sans les risques qu'on peut avoir en C et C++. On sent que le
langage est relativement jeune par rapport à d'autres et qu'il a beaucoup
appris de décennies de développement informatique.
Enfin, son excellente documentation et ses outils bien pratiques ont rendu
ce voyage très agréable !

## Evidemment, tout n'est pas magique

Cependant, il est important de noter que tout n'est pas parfait et que chaque
langage vient avec ses choix, ses forces mais aussi ses inconvénients :

- le code peut être relativement verbeux : le fait de toujours devoir vérifier
que l'opération que l'on souhaite faire soit valide rend le code plus long que dans
d'autres langages.
- le temps de compilation est souvent pointé du doigt : en effet, toutes les
vérifications du langage se font à cette étape.
- toutes ses promesses de sécurité ne sont pas magiques, l'erreur est toujours possible,
même sans code *unsafe*. Par exemple, Cloudflare a réécrit certains de ses services
en Rust pour améliorer leurs performances, les [résultats](https://blog.cloudflare.com/en-us/20-percent-internet-upgrade/)
sont impressionnants : moins de bugs et des améliorations de performance
jusqu'à 25 % ! Cependant, quelques mois plus tard, des milliers de sites ont été
inaccessibles pendant quelques heures à cause d'une [erreur](https://hackaday.com/2025/11/20/how-one-uncaught-rust-exception-took-out-cloudflare/)
dans du code Rust
- parfois le langage n'est pas le mieux adapté : les développeurs de chez Microsoft
expliquent dans [cet article](https://thenewstack.io/microsoft-typescript-devs-explain-why-they-chose-go-over-rust-c/),
pourquoi ils ont préféré porter le compilateur Typescript vers Go au lieu de Rust

## Ressources utiles

Si vous aussi, vous voulez donner une chance à ce langage, voici une petite liste
de ressources que je recommande :

- encore une fois, le [tutoriel officiel](https://doc.rust-lang.org/book/)
- le livre [Effective Rust](https://www.oreilly.com/library/view/effective-rust/9781098151393/)
que j'ai trouvé très complémentaire au tutoriel officiel, très facile à lire
- les replays des conférences de la [Rust Nation UK](https://www.youtube.com/@rustnationuk/videos)
- le livre [Rust for Rustaceans](https://rust-for-rustaceans.com/), pour un public
plus intermédiaire, il faudra notamment que je relise la seconde moitié du livre
quand j'aurai plus d'expérience
- la chaine Youtube [Let's get Rusty](https://www.youtube.com/@letsgetrusty)

## Le mot de la fin

Après ce voyage qui aura duré quelques mois, vais-je continuer à utiliser Rust ?
Oui clairement, la question étant plus de savoir comment car il ne m'est pas utile
au travail. Une de mes passions est également le développement de jeux vidéos mais
je comptais plutôt donner une chance à [Godot](https://godotengine.org/)[^1]
qu'à [Bevy](https://bevy.org/)[^2] prochainement ..
J'ai développé aussi une API avec [Axum](https://github.com/tokio-rs/axum) que je
vais maintenir, et j'avoue que le crate [Ratatui](https://ratatui.rs/) me donne
bien envie pour découvrir le monde des applications qui s'affichent dans le
terminal ! Nous verrons où cela me mène, peut être pour un prochain article !

[^1]: Moteur de jeu open source supportant un langage de programmation maison GDScript
ainsi que C# et C++. Cependant, il existe des bindings Rust ...
[^2]: Moteur de jeu en Rust
