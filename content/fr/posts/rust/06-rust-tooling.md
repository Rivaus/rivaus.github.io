+++
date = '2026-04-10'
draft = false
title = 'Introduction à Rust (6) : des outils bien pensés'
tags = ['rust', 'learn', 'programming']
+++

Notre meilleur ami/outil en Rust est `cargo` un outil en ligne de
commande installé avec le langage. C'est le gestionnaire de paquets
et l’outil de build officiel de Rust.

Pour créer un nouveau projet : `cargo new nom_du_projet`. Cela va créer l'arborescence
suivante :

```bash
nom_du_projet/
├─ src/
│  ├─ main.rs // point d'entrée du programme
├─ Cargo.toml // paramètres du projet + dépendances
```

Pour ajouter une dépendance : `cargo add nom_dependance`
(en Rust on ne parle pas de package ou de librairie externe, mais de *crate*).
Pour vérifier que ça compile : `cargo check`, pour build : `cargo build` et pour
build et lancer l'application : `cargo run`.

Jusque là, rien de nouveau, beaucoup de langages de programmation proposent également
cela.

## Tester

Supposons que vous vouliez faire des tests unitaires, pas besoin d'installer de
dépendances, créez juste un module dédié dans votre ficher :

```rs

fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_happy_path() {
        let res = add(34, 67);

        assert_eq!(res, 34 + 67);
    }
}
```

et lancez `cargo test` pour l'exécuter. Pour faire des tests d'intégration,
créer un dossier `tests` à la racine et créer des fichiers avec des méthodes
annotées par `#[test]`, puis relancer `cargo test`, c'est aussi simple que
cela !

## Formattage et linter

Vous voulez être sûr que tout votre code utilise les conventions d'écriture de
la communauté Rust ? `cargo fmt` et votre code est formaté[^1] !

Vous voulez être sur de ne pas faire des erreurs ou des mauvaises pratiques classiques
du langage, vérifiez votre code avec `cargo clippy`. Clippy possède un grand nombre
de règles de base (+ plein d'autres qu'on peut activer), par exemple :

- des règles de performance :

```rs
let mut v = Vec::new();
v.push(1); // Clippy suggère `Vec::with_capacity(1)` si la taille est connue.
```

- des règles d'anti pattern : `if !condition {...} else {...} // warning inverser
la condition`
- etc...

Beaucoup de langages ont des outils communautaire pour cela mais c'est
la première fois que je les vois intégrés nativement !

## Documentation

Vous voulez générer de la documentation ? Encore facile et prévu dans le langage
nativement ! Commentez vos fonctions avec `///` et vos modules avec `//!` et
lancer `cargo doc` pour générer une documentation au format html ! Ainsi, tous
les *crates* ont une documentation au même format !
Tous les *crates* publics sont disponibles sur [crates.io](https://crates.io/) et
leurs documentations sur [docs.rs](https://docs.rs/). Voici un exemple de documentation
du moteur de jeu Bevy [ici](https://docs.rs/bevy/latest/bevy/).
De plus, les commentaires peuvent contenir du markdown et du code Rust. Pour
vérifier que ce code en commentaire compile, exécutez `cargo test --doc` !

## Conclusion

Au final, rien de magique côté outil en Rust mais le fait que tout soit disponible
sans dépendance externe, sans configuration par défaut rend le langage très
agréable à tester/utiliser ! On arrive à la fin de notre parcours, débrieffons
dans la prochaine [section](07-rust-conclusion.md) !

[^1]: il est possible de configure ce formatage pour rajouter des exceptions/cas customs
