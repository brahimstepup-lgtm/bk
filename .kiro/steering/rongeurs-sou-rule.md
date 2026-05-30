# Rongeurs (Nuisibles > Rongeurs) — deux systèmes séparés

La sous-section **Rongeurs** repose sur **deux systèmes totalement
indépendants**, jamais fusionnés au niveau des indicateurs détaillés.
La comparaison entre les deux n'est autorisée qu'au niveau du
tableau de bord global (totaux uniquement).

## 1. Intérieur usine

- **Sources** : feuilles `pièges mécaniques` + `boîtes à colles`
  (ces deux dispositifs partagent la même légende).
- **Code Sheets** : `D.pests.mech` et `D.pests.glue`.
- **Légende** :

  | Code | Signification        |
  |------|----------------------|
  | SOU  | Souris (capture)     |
  | NT   | Piège non trouvé     |
  | CAS  | Piège cassé          |
  | RT   | Piège retiré         |
  | INAC | Piège inaccessible   |
  | RAS  | Rien à signaler      |

- **Règle de comptage** : seul `SOU` est une capture. `NT`/`CAS`/`RT`/
  `INAC` alimentent l'indicateur séparé de qualité du dispositif. `RAS`
  est neutre.
- **Analyses attendues** :
  - suivi des captures par zone et emplacement,
  - évolution temporelle des captures,
  - comparaison entre pièges mécaniques et boîtes à colles,
  - identification des hotspots à l'intérieur de l'usine,
  - mesure de la pression rongeurs en zone interne
    (taux d'activité = SOU / visites).

## 2. Voie extérieure

- **Source** : feuille `boîtes à raticide` uniquement.
- **Code Sheet** : `D.pests.rat`.
- **Légende** (différente de l'intérieur — appâts toxiques) :

  | Code | Signification         |
  |------|-----------------------|
  | SOU  | Souris (capture)      |
  | EBR  | Boîte ébréchée        |
  | CON  | Appât consommé        |
  | NT   | Boîte non trouvée     |
  | CAS  | Boîte cassée          |
  | RT   | Boîte retirée         |
  | INAC | Boîte inaccessible    |
  | RAS  | Rien à signaler       |

- **Règle de comptage** :
  - `SOU` = capture (souris trouvée morte).
  - `EBR` + `CON` = activité rongeur (appât touché ou consommé) — c'est
    un signal de présence, pas une capture. À suivre dans l'indicateur
    « activité raticide » (séparé de SOU).
  - `NT`/`CAS`/`RT`/`INAC` = qualité du dispositif extérieur.
  - `RAS` neutre.
- **Analyses attendues** :
  - suivi des captures par emplacement extérieur,
  - évolution mensuelle (SOU + EBR + CON empilés),
  - identification des zones extérieures à forte activité,
  - détection des zones critiques récurrentes,
  - analyses spécifiques aux raticides (activité globale = SOU+EBR+CON,
    qualité boîtes), distinctes de l'intérieur.

## Règle de séparation

- Intérieur usine et voie extérieure sont **deux systèmes totalement
  séparés**.
- Aucune fusion directe des légendes : `EBR` et `CON` n'existent que
  côté extérieur.
- Aucun indicateur croisé détaillé. Les hotspots, alertes, taux
  d'activité, qualité dispositif sont calculés indépendamment dans
  chaque sous-section.
- Comparaison autorisée **uniquement** au niveau du tableau de bord
  global (carte « Vue globale rongeurs » en haut de la sous-section
  Rongeurs) : totaux SOU intérieur vs extérieur, évolution mensuelle
  cumulée, donut de répartition.

## Implémentation (front-end uniquement)

Fichier `index.html.txt` :

- `rodentNormalize(v)` — normalise une valeur d'État (majuscules, sans
  accents/ponctuation).
- `rodentCodeIndoor(v)` → `SOU|NT|CAS|RT|INAC|RAS|''`.
- `rodentCodeOutdoor(v)` → `SOU|EBR|CON|NT|CAS|RT|INAC|RAS|''`.
- `rodentGlobalAggregate / KpisHtml / MonthlyChart / CompareChart`
  — vue globale (totaux uniquement).
- `rodentIndoorAggregate / KpisHtml / AlertsHtml / HotspotsHtml /
  PressureHtml / TrapQualityHtml / MonthlyChart / MechVsGlueChart`
  — dashboard intérieur usine, à l'intérieur de `pestsIndoorSub`.
- `rodentOutdoorAggregate / KpisHtml / AlertsHtml / TopPointsHtml /
  CritiquesHtml / TrapQualityHtml / MonthlyChart / ActivityChart`
  — dashboard voie extérieure, à l'intérieur de `pestsRatSub`.
- Orchestrateur `renderPests()` appelle dans l'ordre :
  `renderRodentSouDashboard()` → `renderIndoorRodentDetailed()` →
  `renderOutdoorRodentDetailed()`. Le sélecteur de période
  `#pestsSouPeriodSel` synchronise les trois.

Aucune modification côté `code.txt` (back-end Apps Script) — les
trois feuilles sont déjà exposées dans `D.pests.{rat,mech,glue}`.
