# Guide Martinique (destination)

Ce repo est une **destination** du système multi-guides. Il ne contient que la coquille HTML statique et son contenu — le moteur de rendu (JS/CSS) et l'admin vivent dans [guide-engine](https://github.com/philippebourquin-mq/guide-engine).

Voir `guide-engine/CLAUDE.md` pour l'architecture complète (patterns album, format des sections, protection localStorage, etc.) — ce document ne couvre que ce qui est spécifique à Martinique.

## Fichiers

- `index.html` — coquille statique (header/hero/footer/lightbox), charge `engine.js`/`engine.css` depuis `guide-engine` et appelle `GuideEngine.init(...)`
- `data.json` — contenu du guide (sections, textes, photos)
- GitHub repo : `https://github.com/philippebourquin-mq/martinique.git`

Il n'y a **plus d'admin.html ici** — toute édition se fait depuis l'admin central de `guide-engine` (sélectionner « Martinique » dans le sélecteur de destination).

## Dev local

```bash
python3 -m http.server 4244
# http://localhost:4244/
```

`data.json` local est périmé par rapport à GitHub — ne pas `git pull` sans anticiper l'impact sur les tests.

Note : `index.html` charge le moteur depuis `https://philippebourquin-mq.github.io/guide-engine/engine.js` — même en dev local, c'est la version **publiée** du moteur qui est utilisée (pas de rechargement à chaud si on modifie `guide-engine` en local). Pour tester une modif du moteur avant publication, éditer temporairement l'URL du `<script src>` vers `http://localhost:<port-guide-engine>/engine.js`, tester, puis revenir à l'URL de production avant de commiter.

## Déploiement

```bash
git add index.html data.json && git commit -m "..." && git push
```

GitHub Pages déploie en 2–5 min après le push.

## LocalStorage

| Clé | Contenu |
|-----|---------|
| `guide-site-v1` | Données du guide affichées par `index.html` (propre à ce site, cette origine — même clé utilisée par toutes les destinations, sans risque de collision car localStorage est scopé par origine) |
| `mq_alb_favs` | Favoris utilisateur (photos ★) |

### Protection contre data.json périmé

Le moteur (`guide-engine/engine.js`) ignore le fetch de `data.json` si `stored._ts >= fetched._ts` — logique interne au moteur, rien à faire ici. Voir `guide-engine/CLAUDE.md`.
