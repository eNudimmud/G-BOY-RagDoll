# G*BOY — ragdoll (Three.js + Rapier)

App web statique, autonome, sans backend. Elle tourne hors-ligne une fois construite : three 0.187 et @dimforge/rapier3d-compat 0.21 (WASM embarqué) sont dans le bundle, et le modèle `gboy_rigged.glb` (4,7 Mo, 35 os, textures 2048) est copié dans `public/`.

## Lancer

```bash
cd /workspace/bender/gboy_ragdoll
npm install          # une seule fois (déjà fait sur la box)
npm run build        # -> dist/ (~9,3 Mo)
npx vite preview --port 4173   # ou n'importe quel serveur statique :
python3 -m http.server 8000 -d dist   # puis http://localhost:8000
```

`dist/` se sert depuis n'importe quel serveur statique, avec des chemins relatifs, donc aussi dans un sous-dossier. Il ne s'ouvre pas en `file://`, car le navigateur bloque alors le chargement du .glb.
Développement : `npm run dev`.

## Contrôles (souris + tactile)

- **Glisser sur G*BOY** : on attrape la partie touchée (tête, torse, bras, jambes, pieds) et il suit le pointeur.
- **Toucher / clic bref** : pousse la partie touchée (impulsion dans la direction du regard).
- **Glisser dans le vide** : orbite caméra. **Pincer / molette** : zoom. **Deux doigts / clic droit** : déplacer la caméra.
- **↺ Debout** : il se relève (fondu ~1 s vers la pose de repos, à l'endroit et dans la direction où il est tombé).
- **⤓ Lâcher** : il tombe d'environ 1,6 m avec une rotation aléatoire.
- La caméra suit doucement le bassin.

## Physique

- 13 corps pour le personnage : bassin (boîte), torse (boîte), tête+capuche (sphère r 19 cm), bras, avant-bras+main, cuisses et tibias (capsules), pieds (boîtes). Densité 1000 kg/m³, CCD activé.
- Articulations : rotules pour bassin–torse, cou, épaules et hanches ; charnières limitées pour les coudes (0–137°), les genoux (0–143°) et les chevilles (−34° à +40°). Pas de collision entre segments voisins, mais bras et jambes collisionnent entre eux.
- Les moteurs de rotule de rapier3d-compat 0.21 ne sont pas utilisables : `createImpulseJoint` renvoie un joint générique sans moteurs. « Muscles » PD maison : un léger rappel vers la pose de repos, de l'amortissement et des limites en cône (épaule 130°, hanche 97°, cou/colonne ~45°). Les gains sont mis à l'échelle de l'inertie minimale, donc pas d'explosion.
- **Cape** : 4 chaînes de 3 capsules légères (os `cape_*` du rig), reliées par des rotules souples. Elles collisionnent avec le sol seulement.
- **Repos sans jitter** : quand tout est calme depuis 0,6 s, tout le ragdoll est endormi. Il se réveille au moindre contact, à une poussée ou quand on l'attrape.
- Pas fixe de 1/60 s, 8 itérations de solveur, au plus 4 sous-pas par image.
- Debout = corps cinématiques sur la pose de repos. La première interaction le passe en dynamique.

## Vérification

- `test/shoot.mjs` (Playwright + Chrome headless, SwiftShader) prend les captures du `ragdoll_shots.png`. Il lit le port dans `/tmp/gboy_port` ou dans `PORT=...`.
- `test/interact.mjs` simule de vrais événements souris et tactiles (drag, clic, tap, boutons).
- `test/video.mjs` génère les images de `ragdoll_demo.mp4` (assemblées avec ffmpeg).
- Le mode `?test` met la simulation en pas manuel (`window.__app.advance(s)`). `?test&debug` affiche les colliders en fil de fer.
- Aucune erreur console. Énergie à 0 après la chute (le ragdoll dort).

## Limites connues

- Pas de collision cape ↔ corps : la cape peut traverser les jambes ou le dos dans certaines poses.
- Le relèvement est un fondu cinématique, pas une animation de « get-up ».
- Quand on l'attrape violemment, la cape peut se raidir ou flotter un instant.
- Le bundle JS fait ~5 Mo (1,8 Mo gzip), à cause du WASM Rapier embarqué en base64.
