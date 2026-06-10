# Cartographie interactive des dispositifs — Nuisibles › Rongeurs

Interface ajoutée dans l'onglet **🐀 Nuisibles → section Rongeurs** : un plan
(image de fond) sur lequel sont superposés des **carrés numérotés cliquables**
(boîtes / pièges) + un **sélecteur d'étage** (boutons) pour basculer entre les
3 cartographies. Un clic sur un repère ouvre ses détails + historique.

| Couleur | Famille (`type`) | Feuille Google Sheets interrogée (`source`) |
|---------|------------------|---------------------------------------------|
| 🔵 Bleu   | `ext`  | `boîtes à raticide` (extérieur) → `getDispositifData(num,'rat')` |
| 🟡 Jaune  | `int`  | `pièges mécanique` (intérieur) → `getDispositifData(num,'mech')` |
| 🩷 Rose   | `glue` | `boîtes à colles` (intérieur) → `getDispositifData(num,'glue')` |

Au clic : une **modale** s'ouvre, affiche un **spinner**, appelle la fonction
serveur `getDispositifData(numero, source)` (dans `Code.gs`), puis affiche les
**détails** + l'**historique des passages**.

> **Repli automatique** : si un **piège mécanique** (`'mech'`) n'a aucune ligne
> correspondante, le serveur cherche le **même numéro** dans la feuille
> `boîtes à colles`. Si trouvé, la modale l'indique (badge 🩷 + chip
> « ↪ via boîtes à colles »).

> **Coloration automatique (rose)** : les plans ne dessinent que 2 couleurs
> (bleu ext. / jaune int.). Côté serveur, `getDataForClient` renvoie les numéros
> de chaque feuille intérieure (`pestMechNums`, `pestGlueNums`). À l'affichage,
> un repère **intérieur** dont le n° est dans **boîtes à colles** mais **pas**
> dans **pièges mécanique** est rendu **🩷 rose** (boîte à colle) ; les autres
> restent 🟡 jaunes (piège mécanique). Aucune liste à maintenir à la main :
> déplacez un n° entre les feuilles et la couleur suit au prochain chargement.

---

## 1. Où se trouve le code

| Élément | Fichier | Repère |
|---------|---------|--------|
| Fonction serveur `getDispositifData()` + `trapKey()` | `code.txt` (Code.gs) | après `readPests()` |
| Carte + légende (HTML)        | `index.html.txt` | bloc `<!-- CARTOGRAPHIE INTERACTIVE -->` (section Rongeurs) |
| Modale détail (HTML)          | `index.html.txt` | `<div id="dispoModal">` |
| Styles `.pestmap-*` / `.dispo-*` (CSS) | `index.html.txt` | après `.pests-mini-pane.on` |
| Configuration + logique (JS)  | `index.html.txt` | bloc `CARTOGRAPHIE INTERACTIVE DES DISPOSITIFS` (juste après `renderPests()`) |

---

## 2. Étape 1 — les plans viennent du Sheet (onglet « cartographie nuisibles »)

Les **3 cartographies** (étages) sont lues automatiquement depuis l'onglet
**`cartographie nuisibles`** du Google Sheet :

| zone | lien drive de cartographie |
|------|----------------------------|
| etage -1 | `https://drive.google.com/file/d/…ID…/view` |
| etage0 et voie externe | `https://drive.google.com/file/d/…ID…/view` |
| etage 1 | `https://drive.google.com/file/d/…ID…/view` |

- Colonne **`zone`** = libellé de l'étage (sert aussi de **clé** pour les repères).
- Colonne **`lien drive`** = lien de partage du plan (fichier Drive partagé
  « Tous les utilisateurs disposant du lien »). Le backend (`readCartoNuisibles`)
  en extrait l'ID et l'affiche en image.
- Un **sélecteur de boutons** (un par ligne) apparaît au-dessus de la carte ;
  l'étage dont le libellé contient « etage 0 » est **affiché par défaut**.

> Pour ajouter / changer un plan : éditez le Sheet puis **redéployez** la Web App.
> Tant que l'onglet est vide, un encart d'aide s'affiche.

---

## 3. Étape 2 — positionner les carrés (le plus important)

Les repères sont définis **par étage** dans `PEST_MAP_POINTS_BY_FLOOR`
(`index.html.txt`). La **clé** doit correspondre au libellé `zone` du Sheet :

```js
var PEST_MAP_POINTS_BY_FLOOR = {
  'etage0 et voie externe': [
    { num:'1',  type:'ext', x:9.5,  y:11.0 },   // 🔵 boîte raticide n°1
    { num:'12', type:'int', x:48.5, y:38.5 },   // 🟡 piège mécanique n°12
    …
  ],
  'etage -1': [ /* … */ ],
  'etage 1':  [ /* … */ ]
};
```

- **`num`** : le numéro **affiché** sur le marqueur **ET** recherché dans le Sheet
  (marqueur numéroté **cliquable** → ouvre la fiche détail + historique).
- **`type`** : `'ext'` (bleu) ou `'int'` (jaune) — clés de `PEST_MAP_TYPES`.
- **`x` / `y`** : position en **pourcentage** de l'image (rendu **responsive**).
  `x=0` = bord gauche, `x=100` = bord droit · `y=0` = haut, `y=100` = bas.
  > ⚠️ Les coordonnées dépendent de l'image : **un plan par étage**.
  > Si vous changez l'image d'un étage, **recalibrez** ses repères.

### Méthode rapide : le mode calibrage intégré

1. Ouvrez la Web App, allez sur **Nuisibles → Rongeurs**.
2. Ouvrez la **console** du navigateur (F12) et tapez :

   ```js
   pestMapCalibrate()
   ```

3. **Cliquez** sur l'emplacement d'un piège sur le plan : la console imprime
   une ligne prête à copier, ex. :

   ```js
   { num:'?', type:'ext', x:34.2, y:21.8 },
   ```

4. Remplacez `num` et `type`, collez la ligne dans le tableau de l'étage
   courant : `PEST_MAP_POINTS_BY_FLOOR['<zone de l'étage>']` (la console
   rappelle l'étage actif). Recliquez `pestMapCalibrate()` pour désactiver.

### Méthode manuelle

`x = (distance du bord gauche ÷ largeur de l'image) × 100`
`y = (distance du bord haut ÷ hauteur de l'image) × 100`

Ajoutez/supprimez autant de lignes que de pièges. Aucun autre code à toucher.

---

## 4. Étape 3 — structure attendue du Google Sheets

La fonction `getDispositifData()` **réutilise le lecteur générique
`readPestSheet()`** : la structure des feuilles n'a pas besoin d'être figée.
Pour que la correspondance fonctionne, il suffit que **le numéro du piège
apparaisse dans une colonne d'emplacement / n° piège**.

- Colonnes reconnues automatiquement comme « emplacement / n° » : en-tête
  contenant `zone`, `emplacement`, `lieu`, `secteur`, `piège` ou `boîte`.
- La comparaison se fait sur le **premier groupe de chiffres** trouvé :
  `Piège n°12`, `BAR-12`, `12` → tous reconnus comme **12**.
- Colonnes valorisées si présentes (en-têtes) :
  - **date** → historique chronologique + date d'installation (1ʳᵉ date) ;
  - `produit` / `appât` / `consommable` → champ « Consommable / appât » ;
  - `statut` / `état` / `conformité` → badge de statut ;
  - `action` / `mesure` / `observation` → colonne « Action / Observation ».

Exemple minimal d'onglet `pièges mécanique` :

| Date | N° Piège | Statut | Consommable | Action |
|------|----------|--------|-------------|--------|
| 01/03/2026 | 12 | RAS | — | RAS |
| 15/03/2026 | 12 | SOU | — | Capture retirée, piège réarmé |

→ Clic sur le carré jaune **12** : détails (localisation = `12`, dernier
statut = `SOU`, dernier contrôle = `15/03/2026`, 2 contrôles) + historique
complet trié du plus récent au plus ancien.

---

## 5. Objet JSON renvoyé par `getDispositifData(numero, source)`

```jsonc
{
  "ok": true,
  "found": true,
  "numero": "12",
  "source": "mech",
  "sheetName": "pièges mécanique",
  "typeLabel": "Piège mécanique (intérieur)",
  "headers": ["Date", "N° Piège", "Statut", "Consommable", "Action"],
  "details": {
    "localisation": "12",
    "statut": "SOU",
    "consommable": "",
    "dateInstallation": "01/03/2026",
    "dernierControle": "15/03/2026",
    "nbControles": 2
  },
  "history": [
    { "date":"15/03/2026", "isoDate":"2026-03-15", "statut":"SOU",
      "action":"Capture retirée, piège réarmé",
      "cells":[ {"h":"Date","v":"15/03/2026"}, {"h":"N° Piège","v":"12"}, … ] }
  ]
}
```

`found:false` → aucun n° correspondant (la modale l'indique).
`ok:false` → erreur serveur (message dans `error`).

---

## 6. Les familles de repères (`PEST_MAP_TYPES`)

```js
var PEST_MAP_TYPES = {
  ext:  { label:"Boîte d'appât raticide (extérieur)", color:'blue',   source:'rat'  },
  int:  { label:'Piège mécanique (intérieur)',        color:'yellow', source:'mech' },
  glue: { label:'Boîte à colle (intérieur)',          color:'pink',   source:'glue' }
};
```

- `source` accepte `'rat'`, `'mech'`, `'glue'`, `'fk'` (tous gérés côté serveur).
- Les pièges `'int'` (mécanique) bénéficient du **repli automatique** vers
  `boîtes à colles` (voir l'encart en haut). Pour placer explicitement une
  **boîte à colle** (repère 🩷 rose), utilisez `type:'glue'` dans
  `PEST_MAP_POINTS_BY_FLOOR`.
- Pour une couleur supplémentaire : ajoutez `.pestmap-pt.xxx` (marqueur) et
  `.pestmap-leg.xxx i` (légende) en CSS, puis `color:'xxx'` dans le type.

---

## 7. Cartographie des fly-killers (Insectes volants)

Une **2ᵉ cartographie**, **en bas de la section Insectes volants**, **masquée par
défaut** derrière un bouton **« 🪰 Cartographie des fly-killers »** (clic pour
l'afficher/replier). Même principe que les boîtes d'appât, mais avec des
**icônes en forme d'appareil** (au lieu de carrés) :

| Icône | Type (`type`) | Signification |
|-------|---------------|---------------|
| 🟦 réflecteur + panneau jaune | `fk_glue`    | Fly-killer **avec colle** |
| 🟧 grille électrique          | `fk_nonglue` | Fly-killer **sans colle** |
| ◻️ grisé barré                | `fk_removed` | Fly-killer **retiré** |

- **Plans** : onglet **`cartographie fly-killer`** du Sheet (mêmes colonnes
  `zone` + `lien drive`) → `D.cartoFlyKiller`. Sélecteur d'étage identique
  (Étage -1 / Étage 0 / Étage 1 + brovind ; seul l'étage 0 a un plan pour l'instant).
- **Données au clic** : feuille **`FK`** (insectes volants) — `getDispositifData(num,'fk')`.
- **Repères** : `FLY_MAP_POINTS_BY_FLOOR` (étage 0 pré-rempli : appareils 9→23,
  auto-détectés sur le plan ; bleu = avec colle, orange = sans colle).
- **Icônes** : SVG dans `FLY_ICONS` (`glue` / `nonglue` / `removed`).
  Pour marquer un appareil **retiré**, mettez `type:'fk_removed'`.

Calibrage / ajout des étages -1 et 1 : ajoutez leurs liens Drive dans l'onglet
`cartographie fly-killer`, puis placez les repères dans `FLY_MAP_POINTS_BY_FLOOR`
(même méthode que `pestMapCalibrate()`).
