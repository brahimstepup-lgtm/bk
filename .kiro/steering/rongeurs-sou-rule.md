# Règle d'analyse — Rongeurs (Nuisibles > Rongeurs)

## Codes État (colonne « Critère / État »)

| Code  | Signification         |
|-------|-----------------------|
| NT    | Piège non trouvé      |
| CAS   | Piège cassé           |
| RT    | Piège retiré          |
| INAC  | Piège inaccessible    |
| RAS   | Rien à signaler       |
| SOU   | Souris (capture)      |

## Règle de comptage

Dans la section **Nuisibles > Rongeurs** (sources : *boîtes à raticide*,
*pièges mécanique*, *boîtes à colles*), seule la valeur **`SOU`**
(« Souris ») dans la colonne État doit être comptabilisée comme une
détection / capture de rongeur.

Les codes `NT`, `CAS`, `RT`, `INAC` et `RAS` **ne doivent pas** être
comptés comme captures.

## Indicateurs / visualisations à dériver de SOU

Tous les calculs ci-dessous portent uniquement sur les enregistrements
dont l'État est `SOU` :

1. **Total captures (SOU)** — tous postes confondus.
2. **Top postes de piégeage** — points de contrôle ayant enregistré le
   plus de SOU (champ « emplacement / piège »).
3. **Comparatif Intérieur vs Voie extérieure**
   - Voie extérieure : feuille *boîtes à raticide* (`D.pests.rat`)
   - Intérieur usine : feuilles *pièges mécanique* (`D.pests.mech`)
     + *boîtes à colles* (`D.pests.glue`)
4. **Évolution mensuelle des SOU** (par source et cumulée).
5. **Alerte** : déclenchée lorsqu'un même point de contrôle (zone /
   piège) cumule **≥ 2 SOU sur 30 jours glissants**.
6. **Taux d'activité rongeurs par zone et par période**
   = `SOU(zone, période) / total_visites(zone, période)`.

## Indicateur séparé — Qualité du dispositif de piégeage

Les codes `NT`, `CAS`, `RT`, `INAC` ne sont **pas** des captures mais
doivent être suivis dans un **indicateur séparé de qualité du dispositif**
afin d'identifier les pièges manquants, détériorés, retirés ou
inaccessibles. Le code `RAS` reste neutre (visite normale sans capture
ni anomalie).

## Implémentation

- Front-end : `index.html.txt`
  - Helper `rodentEtatCode(v)` normalise l'État vers
    `SOU | NT | CAS | RT | INAC | RAS | ''`.
  - Bloc `pestsRodentDashSub` (en tête de la sous-section Rongeurs)
    affiche les KPI SOU, les graphiques, les alertes, le taux
    d'activité par zone et l'indicateur de qualité du dispositif.
  - Les sous-sections existantes (raticide, mécanique, colles)
    affichent en plus un chip « Captures SOU ».
- Back-end : `code.txt` — aucune modification structurelle, l'État
  arrive déjà dans `entry.statut` via la détection des entêtes
  contenant « etat » / « statut ».
