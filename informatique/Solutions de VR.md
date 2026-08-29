---
schema_version: 1
uid: "01M02JG1VGQC2EGWCC5VZ356VA"
titre: "Solutions de VR"
aliases:
  - "WebXR"
  - "Réalité virtuelle sur le web"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - realite-virtuelle
  - developpement-web
  - threejs
  - webxr
resume: "Panorama daté (août 2026) des solutions de réalité virtuelle et augmentée sur le web : standard WebXR et navigateurs compatibles, moteurs three.js, Babylon.js, A-Frame et PlayCanvas, mondes sociaux (Hubs Community Edition, Hyperfy, Voxels), formats glTF et USDZ, casques, et sort des liens transmis en 2023."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2023-02-08
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Solutions de VR

> [!abstract] Objectif
> Savoir par où commencer pour produire une expérience de réalité virtuelle ou augmentée **dans le navigateur**, sans boutique d'applications : le standard WebXR, les moteurs JavaScript à choisir selon le niveau d'abstraction voulu, les plateformes de mondes partagés encore vivantes, les formats de scènes et les casques qui affichent tout cela en 2026.

> [!info] État de la fiche
> Établie le 29 août 2026 à partir de la liste de sept liens transmise par Vincent en février 2023, dont deux ne mènent plus nulle part : Mozilla a arrêté Hubs le 31 mai 2024 et Third Room a été abandonné en 2023. Versions vérifiées sur npm : three.js r185, A-Frame 1.8, Babylon.js 9.

Voir aussi : [[Javascript]], [[HTML]], [[Outils IA]], [[SOLID|Solid (Social Linked Data)]].

# 1. Le socle : WebXR

**WebXR Device API** est le standard du W3C qui donne à une page web l'accès aux casques et aux capteurs (pose de la tête et des mains, contrôleurs, suivi des mains, ancrages en réalité augmentée). Une session est demandée par `navigator.xr.requestSession("immersive-vr")` ou `"immersive-ar"` ; le rendu passe par WebGL ou, de plus en plus, **WebGPU**.

Navigateurs compatibles en 2026 : le navigateur des casques Meta Quest (Horizon OS), Chrome et Edge sur Android et sur Android XR, Safari sur Apple Vision Pro (visionOS), **Wolvic** (navigateur libre d'Igalia pour casques), Firefox et Chrome sur ordinateur avec un casque PC. Sans casque, tous les moteurs ci-dessous retombent sur un rendu 3D classique dans l'onglet, ce qui rend les projets démontrables en classe.

# 2. Moteurs et bibliothèques

| Solution | Niveau | Pour qui | Points clés |
|---|---|---|---|
| **three.js** (r185) | bibliothèque 3D bas niveau | contrôle total, intégration dans une application existante | référence du 3D web, rendu WebGPU, chargeurs glTF, `WebXRManager` ; React Three Fiber pour l'écosystème React |
| **Babylon.js** (9.x) | moteur complet | applications riches, physique, interface, outils | éditeur en ligne (Playground, Sandbox), WebGPU, XR de série ; soutenu par Microsoft |
| **A-Frame** (1.8) | déclaratif, balises HTML | prototypage, enseignement, débutants | `<a-scene>`, `<a-box>`, composants réutilisables ; construit sur three.js ; une scène VR tient en dix lignes |
| **PlayCanvas** | moteur avec éditeur visuel | équipes, jeux | moteur open source, éditeur collaboratif en ligne |
| **Godot 4** / **Unity** / **Unreal** | moteurs natifs | projets d'envergure | export web possible (Godot, Unity) mais fichiers lourds ; WebXR mieux servi par les moteurs web ci-dessus |

Règle simple : **A-Frame** pour apprendre et prototyper, **three.js** pour maîtriser, **Babylon.js** pour un produit complet.

# 3. Mondes partagés et sociaux

| Plateforme | État en 2026 |
|---|---|
| **Mozilla Hubs** | service arrêté le 31 mai 2024 ; le code vit dans la **Hubs Community Edition**, maintenue par la Hubs Foundation, à héberger soi-même |
| **Hyperfy** | moteur de mondes publié en open source (v2, 2025), monde hébergé ou auto-hébergé |
| **Voxels** (ex-Cryptovoxels) | monde persistant fondé sur des parcelles adossées à une blockchain ; encore en ligne |
| **Third Room** | projet Matrix/WebXR d'Element, abandonné en 2023 |
| **VRChat**, **Rec Room**, **Horizon Worlds** | applications natives dominantes, hors web |

Pour un usage pédagogique ou associatif, une instance Hubs Community Edition ou Hyperfy auto-hébergée reste la voie la plus ouverte : pas de compte tiers, pas de boutique, un simple lien à partager.

# 4. Formats et chaîne de production

- **glTF 2.0 / GLB** (Khronos) : le format d'échange des scènes web, chargé nativement par tous les moteurs ; exporté par Blender.
- **USDZ** (Apple) : requis pour la réalité augmentée sur iOS et visionOS ; conversion depuis glTF avec les outils d'Apple ou de Blender.
- **OpenXR** : le standard natif côté casques, dont WebXR est l'équivalent web.
- **Blender** pour modéliser, **Spoke** (Hubs) ou les éditeurs Babylon/PlayCanvas pour assembler des scènes ; les générateurs 3D par IA (voir [[Outils IA]]) accélèrent le prototypage d'assets.
- Tendance : les **nuages gaussiens** (*Gaussian splatting*) capturés au téléphone se visualisent désormais dans three.js et Babylon.js, ce qui permet de reproduire un lieu réel en quelques minutes.

# 5. Matériel

Casques les plus répandus en 2026 : Meta Quest 3 et 3S (Horizon OS), Apple Vision Pro (visionOS), casques Android XR (Samsung Galaxy XR), PICO 4 Ultra, casques PC (Valve Index, Bigscreen). Tous disposent d'un navigateur WebXR ; le Quest reste le choix économique pour une salle de classe, le Vision Pro celui de la réalité augmentée de qualité.

# 6. Premier essai en dix lignes

```html
<!doctype html>
<html>
  <head>
    <script src="https://aframe.io/releases/1.8.0/aframe.min.js"></script>
  </head>
  <body>
    <a-scene>
      <a-box position="-1 0.5 -3" rotation="0 45 0" color="#4CC3D9"></a-box>
      <a-sphere position="0 1.25 -5" radius="1.25" color="#EF2D5E"></a-sphere>
      <a-plane position="0 0 -4" rotation="-90 0 0" width="8" height="8" color="#7BC8A4"></a-plane>
      <a-sky color="#ECECEC"></a-sky>
    </a-scene>
  </body>
</html>
```

Ouvrir la page sur un casque et cliquer sur le bouton VR : la scène s'affiche en immersion ; sur un ordinateur, elle se manipule à la souris. Servir la page en HTTPS (obligatoire pour WebXR hors `localhost`).

# 7. Sources

- WebXR Device API : <https://www.w3.org/TR/webxr/> ; guide MDN : <https://developer.mozilla.org/docs/Web/API/WebXR_Device_API>
- three.js : <https://threejs.org/> ; Babylon.js : <https://www.babylonjs.com/> ; A-Frame : <https://aframe.io/> ; PlayCanvas : <https://playcanvas.com/>
- Hubs Foundation : <https://hubsfoundation.org/> ; Hyperfy : <https://hyperfy.io/> ; Voxels : <https://www.voxels.com/>
- Wolvic : <https://wolvic.com/>
- glTF : <https://www.khronos.org/gltf/> ; OpenXR : <https://www.khronos.org/openxr/>
- Fin de Mozilla Hubs (31 mai 2024) : <https://support.mozilla.org/fr/kb/fin-prise-en-charge-mozilla-hubs>
