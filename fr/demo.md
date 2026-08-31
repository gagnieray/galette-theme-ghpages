---
ref: doc
title: Référence des éléments
---

Tous les éléments mis en forme par le thème, sur une seule page, pour juger
d'une modification d'un coup d'œil. Cette page existe en anglais et en
français, ce qui permet de vérifier le sélecteur de langue.

## Titres

### Troisième niveau

#### Quatrième niveau

## Texte

Un paragraphe avec du **gras**, de l'*italique*, du `code en ligne`, un
[lien vers galette.eu](https://galette.eu) et une phrase assez longue pour
passer à la ligne sur un écran étroit, afin de juger honnêtement de la
justification.

> Une citation, pour les notes et avertissements dont une documentation de
> plugin finit toujours par avoir besoin.

## Listes

* premier élément
* deuxième élément, avec une liste imbriquée :
  * imbriqué un
  * imbriqué deux
* troisième élément

1. numéroté
2. encore numéroté

## Badges de téléchargement

<div class="download-badge" markdown="1">
[![Obtenir la dernière version](https://img.shields.io/badge/2.2.1-Fullcard-ffb619.svg?logo=php&logoColor=white&style=for-the-badge)](https://galette.eu/download/plugins/)
[![Obtenir la nightly](https://img.shields.io/badge/nightly-Fullcard-ffb619.svg?logo=php&logoColor=white&style=for-the-badge)](https://galette.eu/download/plugins/)
</div>

## Code

```bash
$ cd /var/www/html/galette/plugins
$ wget https://galette.eu/download/plugins/galette-plugin-fullcard-2.2.1.tar.bz2
$ tar xjvf galette-plugin-fullcard-2.2.1.tar.bz2
```

## Tableau

Les tableaux larges défilent dans leur propre conteneur, pour que la page
elle-même ne défile jamais latéralement — les entourer de
`<div class="table-wrapper" markdown="1">`.

<div class="table-wrapper" markdown="1">

| Réglage | Défaut | Description |
| --- | --- | --- |
| `maintainer` | `community` | La pastille affichée dans l'en-tête |
| `plugin.archive` | — | Nom de base de l'archive de version |
| `plugin.min_galette` | — | Version minimale de Galette prise en charge |

</div>

## Filet horizontal

---

Et c'est tout.
