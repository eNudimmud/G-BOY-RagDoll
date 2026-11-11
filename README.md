# G*BOY — ragdoll (Three.js + Rapier)

App web statique et autonome, sans backend. Une fois construite, elle tourne hors ligne : three 0.187 et @dimforge/rapier3d-compat 0.21 (WASM embarqué) sont dans le bundle. Le modèle `gboy_pieces.glb` (2,4 Mo, 65 os) est copié dans `public/`.

En ligne : https://enudimmud.github.io/G-BOY-RagDoll/

## Le personnage : 10 pièces séparées (v2.1, 11.11.2026, passe de fidélité sur la planche de JD)

Reconstruit dans Blender comme un vrai personnage de jeu, en suivant la planche de pièces de JD. Chaque pièce a son maillage propre, ses UV et son matériau. Sources : `/workspace/bender/gboy_hq/pieces/` (`build_pieces.py`, `rig_pieces.py`, `render_*.py`).

| # | Pièce | Construction |
|---|---|---|
| 01 | tête | sphère noire mate |
| 02 | capuche à oreilles | coque feutre extraite de la forme Tripo et nettoyée, **bord roulé épais autour du visage**, ourlet arrondi en bas (plus de liseré visible), intérieur rouge, oreilles en cônes à pointe arrondie |
| 03 | cape + nœud | **un seul objet** : nappe unique (72×45 quads) ajustée à la silhouette Tripo, plis verticaux, ourlet ondulé, rouge dedans et dehors ; **grand nœud sous le menton** (boucles larges, pans jusqu'à mi-poitrine, bouts évasés) |
| 04 | torse | noir, épaules tombantes, membres fins |
| 05 | t-shirt moutarde (#E8B83A) | col rond **côtelé**, manches courtes à ourlet, ourlet droit aux hanches |
| 06 | short chino (#CDB894) | ceinture, **8 passants**, bouton, **poches en biais**, larges revers roulés |
| 07 | mains ×2 | noires, doigts ronds légèrement repliés, pose détendue |
| 08 | chaussettes crème (#EEE6D6) ×2 | côtes |
| 09 | sneakers lavande (#A68BD8) ×2 | forme et texture projetées depuis le Tripo puis teintées en lavande : semelle et bout blancs, lacets blancs, **petit écusson blanc sur le côté** |
| 10 | yeux | ovales blancs émissifs, grands (environ 1/4 de la largeur du visage), à plat |

Palette : noir, rouge #C8202A, moutarde, beige, crème, lavande.

Rendus : `pieces_turnaround.png` (face, 3/4, profil, dos) et `pieces_exploded.png` (vue éclatée).

Squelette : même squelette que la v1 (hips … toe) plus **7 chaînes de cape × 6 os**. Le corps est en poids automatiques nettoyés. Les vêtements reprennent les poids du corps, la capuche est rigide sur la tête avec un fondu vers le buste en bas, les mains sont sur `hand`, les baskets sur `foot`/`toe`, et la cape a des poids bilinéaires sur les chaînes, et le nœud suit le buste. Les poids du t-shirt et du short sont lissés (6 passes) pour éviter le froissement au col et aux épaules.

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
- **Anti-repli** : les rangées hautes gardent la forme du feutre, des particules non voisines restent à au moins 6,5 cm les unes des autres (auto-évitement), et chaque chaîne pend toujours vers les pieds par rapport au torse. La cape ne se replie donc plus par-dessus la tête quand il est couché.
- Repos sans tremblement : le ragdoll entier s'endort après 0,6 s de calme.
- Pas fixe de 1/60 s, au plus 4 sous-pas par image.

## Rendu

- Tone mapping Neutral, couleurs de base prises sur les références de JD (rouge feutre, jaune moutarde, beige, crème, violet lavande). Les yeux ne passent pas par le tone mapping, donc ils restent blancs purs.
- Micro-reliefs générés au chargement (feutre, coton, côtes des chaussettes), sans fichier supplémentaire.

## Vérification

- `test/polish.mjs` + `test/sheet.py` : planche de 18 gros plans, avec les angles signalés par JD (couché, vu d'en bas, de dos, sous la cape, intérieur de la capuche, tenu par la tête, le pied ou le bras, après « Debout »). Résultats dans `polish_before.png` (v1), `polish_after.png` (v2) et `polish_after2.png` (v2.1).
- `test/close.mjs` : très gros plans de la nuque et de la capuche, debout et couché.
- `test/video.mjs` : images de `ragdoll_demo.mp4`. `test/interact.mjs` : vrais événements souris et tactiles. `test/jitter.mjs` : vitesses au repos.
- Le mode `?test` passe en pas manuel (`window.__app.advance(s)`), et `?test&debug` affiche les colliders.

## Limites connues

- L'auto-évitement de la cape se fait entre particules, pas entre triangles : un léger chevauchement reste possible dans les plis serrés.
- Si on le tient par un pied, la cape retombe naturellement sur la tête.
- Le relèvement est un fondu cinématique, pas une vraie animation.
- Les vêtements sont des coques fines (4 mm) : dans les poses extrêmes, on peut voir un léger froissement aux épaules et au col.
- Le bundle JS fait environ 5 Mo (1,8 Mo gzip), à cause du WASM Rapier embarqué.
