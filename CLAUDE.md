# Guide Martinique

Guide de voyage statique + panel admin → GitHub Pages.

## Fichiers

- `index.html` — viewer public (tout le JS inline, ~47k chars)
- `admin.html` — panel admin (publie via GitHub API)
- `data.json` — données (servi localement par Python http.server)
- GitHub repo : `https://github.com/philippebourquin-mq/martinique.git`

## Dev local

```bash
python3 -m http.server 4244
# http://localhost:4244/index.html
# http://localhost:4244/admin.html
```

`data.json` local est périmé par rapport à GitHub — ne pas `git pull` sans anticiper l'impact sur les tests.

## Déploiement

```bash
git add index.html admin.html && git commit -m "..." && git push
# Si rejeté (admin a publié entre-temps) :
git pull --rebase && git push
```

GitHub Pages déploie en 2–5 min après le push.

## LocalStorage

| Clé | Contenu |
|-----|---------|
| `mtq-guide-v1` | Données du guide (partagée entre index et admin) |
| `mtq-admin-session` | Session admin (connecté/déconnecté) |
| `mtq-gh-config` | Config GitHub (owner, repo, token) |
| `mq_alb_favs` | Favoris utilisateur (photos ★) |

### Protection contre data.json périmé

`admin.saveToGitHub()` injecte `data._ts = Date.now()` avant chaque publication.

`index.html` ignore le résultat du fetch si :
```javascript
stored && (stored._ts || 0) >= (fetched._ts || 0)
```
**Ne jamais supprimer cette protection.** Le `data.json` local n'est jamais mis à jour par l'admin (Python http.server sert le fichier local statique). Sans cette règle, Cmd+Shift+R écrase les formats publiés.

## Patterns album

`_ALB_PAT` : 12 patterns cyclés (duo / trio / quad / portrait vertical).
`_CLS_PAT` : `_ALB_PAT` filtré sans `bigFirst`/`bigLast` (sections classiques — CSS incompatible).

Fallbacks indépendants du tableau (résistants aux réordres) :
- `_PAT_SOLO` — 1 photo pleine largeur
- `_PAT_DUO`  — 2 photos égales
- `_PAT_TRIO` — 3 photos égales

**Ne jamais référencer `_ALB_PAT[i]` par indice hardcodé** — utiliser ces constantes.

### `_albSz(photo)` — catégories de layout forcé

| `photo.layout` | Catégorie | Comportement |
|----------------|-----------|--------------|
| `null`, `'1x1'`, `'default'` | `null` (auto) | Intégré dans le cycle |
| `'1x3'`, `'2x3'` | `'solo'` | Pleine largeur isolé |
| `'2x1'` | `'tall'` | Portrait vertical, groupé avec 1–2 autos suivantes |
| `'1x2'`, `'2x2'` | `'large'` | Grand format, groupé avec 1 auto suivante |

### Anti-orphelin (look-ahead)

Si prendre `n` photos laisserait exactement 1 orphelin : ajuster `n` (élargir si ≤3, rétrécir sinon).

## Sections

### Types

- `classic` — blocs texte/photo avec layout CSS grid (`block-col-2`, `block-row-2`)
- `album`   — galerie photo avec patterns Cheerz/Lalalab

### Rendu classique hybride (`renderClassicSection`)

- Section **tout auto** (`layout` absent ou `1x1`) → cycling album avec `_CLS_PAT`
- Section **mixte** (au moins un layout forcé) → CSS grid `repeat(3,1fr)` natif

Les classes `block-col-2` (span 2 colonnes) et `block-row-2` (span 2 rangées) ne fonctionnent qu'en mode mixte (grid natif).

## Admin — picker de format

6 formats disponibles pour les photos album ET les blocs classiques :
`1x1` / `1x2` / `1x3` / `2x1` / `2x2` / `2x3`

Icônes rectangles SVG proportionnels (pas de texte) — identiques dans les deux contextes.

`1x1` = pas de `layout` sur l'objet (supprimé). Les autres = `photo.layout = 'Nx...'`.
