# Publier Doppler

Le marketplace d'OpenIDEA liste les plugins communautaires à travers le registre
[l0ng-ai/openidea-plugins](https://github.com/l0ng-ai/openidea-plugins).
Le plugin reste dans **ton** dépôt. Le registre stocke seulement un pointeur
vers un paquet signé (son URL et son SHA-256).

```text
ton dépôt GitHub ──release──▶ .oiplugin signé
                                 │ URL + sha256
                                 ▼
             PR dans l0ng-ai/openidea-plugins ──CI + revue──▶ marketplace (< 1 h)
```

## 0. Une seule fois : l'outil et la clé

```sh
# L'outil officiel, à la version de ton OpenIDEA (0.1.50)
cargo install --locked --git https://github.com/l0ng-ai/openidea --tag v0.1.50 ide-host --bin openidea-plugin

# Ta clé d'éditeur (Ed25519). Elle affiche la clé PUBLIQUE, à noter.
# Garde ce fichier HORS du dépôt et sauvegarde-le (gestionnaire de mots de passe).
openidea-plugin keygen ~/.config/openidea/rhombicdodecahedron.key
```

Celui qui possède ce fichier peut publier en ton nom. Si tu le perds, tu ne
peux plus signer de mises à jour sans passer par une révocation.

## 1. Vérifier le plugin

```sh
cd openidea-doppler
openidea-plugin check .
```

Puis installe-le en local (**Settings › Plugins › ⚙ › Install Plugin from
Disk…**) et parcours la checklist de la doc :

- la fenêtre Project avec une sélection ;
- l'éditeur avec plusieurs langages ;
- Search Everywhere, un menu popup, une info-bulle et les Settings ;
- le terminal (`ls --color`), un diff et le Commit tool window.

C'est le bon moment pour remplacer `assets/*.png` par de **vraies captures**
d'OpenIDEA : celles du dépôt viennent des maquettes.

## 2. Mettre le code sur GitHub (dépôt public obligatoire)

```sh
git init && git add . && git commit -m "Doppler 0.1.0"
gh repo create Rhombicdodecahedron/openidea-doppler --public --source . --push
```

Le registre exige un dépôt public sous licence open source. C'est déjà prévu
avec MIT dans `LICENSE`.

## 3. Construire, signer et publier une release

```sh
openidea-plugin pack . --key ~/.config/openidea/rhombicdodecahedron.key --publisher rhombicdodecahedron
# produit rhombicdodecahedron.doppler-0.1.0.oiplugin

shasum -a 256 rhombicdodecahedron.doppler-0.1.0.oiplugin   # à noter
gh release create v0.1.0 rhombicdodecahedron.doppler-0.1.0.oiplugin \
  --title "Doppler 0.1.0" --notes "Crimson Cobalt et Neon Siren."
```

Le paquet contient `plugin.toml`, `icon.svg`, `themes/`, `README.md` et
`LICENSE`. Le dossier `assets/` et ce guide n'y sont pas inclus.

## 4. Pull request dans le registre

Fork `l0ng-ai/openidea-plugins` puis ajoute deux fichiers.

`publishers/rhombicdodecahedron.toml` (seulement la première fois) :

```toml
name = "Rhombicosidodecahedron"
owners = ["Rhombicdodecahedron"]
keys = ["<clé publique affichée par keygen>"]
revoked-keys = []
```

`plugins/rhombicdodecahedron.doppler.toml` :

```toml
repository = "https://github.com/Rhombicdodecahedron/openidea-doppler"

[[versions]]
version = "0.1.0"
package = "https://github.com/Rhombicdodecahedron/openidea-doppler/releases/download/v0.1.0/rhombicdodecahedron.doppler-0.1.0.oiplugin"
sha256 = "<sha256 de l'étape 3>"
```

Ensuite, la CI télécharge le paquet, vérifie le SHA-256 et la signature, et
lance `openidea-plugin check`. Un mainteneur relit (c'est ta première
publication). Une fois la PR fusionnée, le plugin apparaît dans **Settings ›
Plugins › Marketplace** en moins d'une heure.

## Mises à jour

1. Change les couleurs et augmente `version` et `change-notes` dans `plugin.toml`.
2. Refais l'étape 3 avec le nouveau numéro, par exemple `v0.2.0`.
3. Ouvre une PR dans le registre qui **ajoute** un bloc `[[versions]]`, en
   gardant l'ancien.

Les utilisateurs voient alors « Update All » dans l'IDE dans les 24 h.

## Alternative sans marketplace

Joindre le `.oiplugin` à une release GitHub suffit pour le partager. Les gens
l'installent avec **Install Plugin from Package File…**. Ils ne recevront pas
les mises à jour automatiques, et un paquet non signé affiche un avertissement.
