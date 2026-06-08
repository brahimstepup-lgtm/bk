# Cartographie interactive des dispositifs — Nuisibles › Rongeurs

Interface ajoutée dans l'onglet **🐀 Nuisibles → section Rongeurs** : un plan
(image de fond) sur lequel sont superposés des **carrés cliquables**.

| Couleur | Famille (`type`) | Feuille Google Sheets interrogée (`source`) |
|---------|------------------|---------------------------------------------|
| 🔵 Bleu   | `ext` | `boîtes à raticide` (extérieur) → `getDispositifData(num,'rat')` |
| 🟡 Jaune  | `int` | `pièges mécanique` (intérieur) → `getDispositifData(num,'mech')` |

Au clic : une **modale** s'ouvre, affiche un **spinner**, appelle la fonction
serveur `getDispositifData(numero, source)` (dans `Code.gs`), puis affiche les
**détails** + l'**historique des passages**.

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

## 2. Étape 1 — mettre votre plan en image de fond

1. Mettez votre image de plan sur **Google Drive**, clic droit →
   **Partager → Tous les utilisateurs disposant du lien** (Lecteur).
2. Récupérez l'**ID** du fichier dans son URL :
   `https://drive.google.com/file/d/`**`1AbCdEf…ID…`**`/view`
3. Dans `index.html.txt`, renseignez la constante :

   ```js
   var PEST_MAP_IMG_URL = 'https://drive.google.com/thumbnail?id=1AbCdEf…ID…&sz=w2000';
   ```

   > Tant que cette constante est vide, un encart d'aide s'affiche à la place
   > du plan (le reste de l'app fonctionne normalement).

Vous pouvez aussi utiliser n'importe quelle URL d'image publique (`https://…/plan.png`).

---

## 3. Étape 2 — positionner les carrés (le plus important)

Chaque repère est **une ligne** dans le tableau `PEST_MAP_POINTS` :

```js
var PEST_MAP_POINTS = [
  { num:'1',  type:'ext', x:9.5,  y:11.0 },   // 🔵 boîte raticide n°1
  { num:'12', type:'int', x:48.5, y:38.5 },   // 🟡 piège mécanique n°12
  …
];
```

- **`num`** : le numéro **affiché** dans le carré **ET** recherché dans le Sheet.
- **`type`** : `'ext'` (bleu) ou `'int'` (jaune) — clés de `PEST_MAP_TYPES`.
- **`x` / `y`** : position en **pourcentage** de l'image.
  `x=0` = bord gauche, `x=100` = bord droit · `y=0` = haut, `y=100` = bas.
  *(Les % garantissent un rendu **responsive** : les carrés suivent l'image
  quelle que soit la taille de l'écran.)*

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

4. Remplacez `num` et `type`, collez la ligne dans `PEST_MAP_POINTS`.
   Recliquez `pestMapCalibrate()` pour désactiver le mode.

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

## 6. Ajouter une 3ᵉ famille (ex. boîtes à colles)

```js
var PEST_MAP_TYPES = {
  ext:  { label:"Boîte d'appât raticide (extérieur)", color:'blue',   source:'rat'  },
  int:  { label:'Piège mécanique (intérieur)',        color:'yellow', source:'mech' },
  glue: { label:'Boîte à colle (intérieur)',          color:'yellow', source:'glue' }
};
```

`source` accepte `'rat'`, `'mech'`, `'glue'`, `'fk'` (déjà gérés côté serveur).
Pour une couleur supplémentaire, ajoutez une classe `.pestmap-pt.xxx` en CSS.
