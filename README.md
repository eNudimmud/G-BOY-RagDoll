# G*BOY — ragdoll (Three.js + Rapier)

App web statique et autonome, sans backend. Une fois construite, elle tourne hors ligne : three 0.187 et @dimforge/rapier3d-compat 0.21 (WASM embarqué) sont dans le bundle. Le modèle `gboy_pieces.glb` (2,3 Mo, 65 os) est copié dans `public/`.

En ligne : https://enudimmud.github.io/G-BOY-RagDoll/

## Le personnage : 10 pièces séparées (v2, 11.11.2026)

Reconstruit dans Blender comme un vrai personnage de jeu, en suivant la planche de pièces de JD. Chaque pièce a son maillage propre, ses UV et son matériau. Sources : `/workspace/bender/gboy_hq/pieces/` (`build_pieces.py`, `rig_pieces.py`, `render_*.py`).

| # | Pièce | Construction |
|---|---|---|
| 01 | tête | ellipsoïde noire lisse |
| 02 | capuche à oreilles | coque extraite de la forme Tripo, nettoyée (plus de collerette inventée dans le dos, bords lissés, sans dents ni trous), épaisseur 8 mm, **intérieur rouge feutre propre** |
| 03 | cape + nœud | **une seule nappe** (72×45 quads) ajustée à la silhouette Tripo, ourlet ondulé, double face rouge feutre ; le nœud (boucles, pans, cordon au cou) est modélisé |
| 04 | corps | noir, bras longs, maillage « skin » propre |
| 05 | t-shirt jaune | coque décalée du corps, ourlets droits |
| 06 | short beige | revers roulés, bouton |
| 07 | mains ×2 | noires, séparées, 5 doigts |
| 08 | chaussettes crème ×2 | côtes (normal map générée) |
| 09 | baskets violettes ×2 | forme et texture (semelle blanche, lacets) projetées depuis le Tripo, marge 24 px |
| 10 | yeux | ovales blancs émissifs, à plat sur le visage, sans iris |

Rendus : `pieces_turnaround.png` (face, 3/4, profil, dos) et `pieces_exploded.png` (vue éclatée).

Squelette : même squelette que la v1 (hips … toe) plus **7 chaînes de cape × 6 os**. Le corps est en poids automatiques nettoyés. Les vêtements reprennent les poids du corps, la capuche est rigide sur la tête avec un fondu vers le buste en bas, les mains sont sur `hand`, les baskets sur `foot`/`toe`, et la cape a des poids bilinéaires sur les chaînes.

## Lancer

```bash
cd /workspace/bender/gboy_ragdoll
npm install          # une seule fois
npm run build        # -> dist/
python3 -m http.server 8000 -d dist   # puis http://localhost:8000
```

Ne s'ouvre pas en `file://`, car le navigateur bloque alors le chargement du .glb. Pour développer : `npm run dev`.

## Contrôles (souris et tactile)

- **Glisser sur G*BOY** : il est attrapé par la partie touchée (tête, torse, bras, jambes, pieds).
- **Toucher ou clic bref** : pousse la partie touchée.
- **Glisser dans le vide** : orbite de la caméra. **Pincer ou molette** : zoom. **Deux doigts ou clic droit** : déplacer la caméra.
- **↺ Debout** : il se relève à l'endroit où il est tombé, **toujours face à la caméra**.
- **⤓ Lâcher** : chute d'environ 1,6 m avec une rotation aléatoire.

## Physique

- 13 corps Rapier pour le personnage (bassin, torse, tête, bras, avant-bras, cuisses, tibias, pieds), avec des « muscles » PD maison et des limites en cône. Le cou est limité à environ 29° pour éviter que la capuche se plie de façon laide.
- **Cape = tissu Verlet** (`src/cloth.js`) : 42 particules placées aux articulations des 7 chaînes. La rangée du haut est épinglée au torse. Contraintes de structure, de cisaillement et de flexion, plus une légère mémoire de forme pour le côté feutre. **Collision avec tous les colliders du corps** (boîtes, capsules, sphère) : les particules, et des points le long de chaque bord, sont repoussés à 3 cm du corps. Elle collisionne aussi avec le sol. Elle s'endort quand tout est calme, donc zéro tremblement au repos.
- Repos sans tremblement : le ragdoll entier s'endort après 0,6 s de calme.
- Pas fixe de 1/60 s, au plus 4 sous-pas par image.

## Rendu

- Tone mapping Neutral, couleurs de base prises sur les références de JD (rouge feutre, jaune moutarde, beige, crème, violet lavande). Les yeux ne passent pas par le tone mapping, donc ils restent blancs purs.
- Micro-reliefs générés au chargement (feutre, coton, côtes des chaussettes), sans fichier supplémentaire.

## Vérification

- `test/polish.mjs` + `test/sheet.py` : planche de 18 gros plans, avec les angles signalés par JD (couché, vu d'en bas, de dos, sous la cape, intérieur de la capuche, tenu par la tête, le pied ou le bras, après « Debout »). Résultats dans `polish_before.png` (v1 en ligne) et `polish_after.png` (v2).
- `test/close.mjs` : très gros plans de la nuque et de la capuche, debout et couché.
- `test/video.mjs` : images de `ragdoll_demo.mp4`. `test/interact.mjs` : vrais événements souris et tactiles. `test/jitter.mjs` : vitesses au repos.
- Le mode `?test` passe en pas manuel (`window.__app.advance(s)`), et `?test&debug` affiche les colliders.

## Limites connues

- La cape n'a pas d'auto-collision : couchée, elle peut se replier à travers elle-même.
- Le relèvement est un fondu cinématique, pas une vraie animation.
- Les vêtements sont des coques fines (4 mm) : dans les poses extrêmes, on peut voir un léger froissement aux épaules et au col.
- Le bundle JS fait environ 5 Mo (1,8 Mo gzip), à cause du WASM Rapier embarqué.
