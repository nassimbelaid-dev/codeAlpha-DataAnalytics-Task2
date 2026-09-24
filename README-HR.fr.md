# 👥 Tableau de Bord Analytique RH — Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-217346?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Star Schema](https://img.shields.io/badge/Data%20Modeling-Role--Playing%20Dimension-blue?style=for-the-badge)
![HR Analytics](https://img.shields.io/badge/HR%20Analytics-Turnover%20%26%20Attrition-purple?style=for-the-badge)
![AI Visuals](https://img.shields.io/badge/Power%20BI-Key%20Influencers%20%2F%20Decomposition%20Tree-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge)

Tableau de bord Power BI permettant de piloter le recrutement, le turnover, la satisfaction/performance et le risque d'attrition d'une entreprise, à partir du jeu de données *Human Resources Data Set* (HRDataset_v14).

---

## 🎯 Objectif du projet

Donner à une fonction RH une vision claire et actionnable de la main-d'œuvre, structurée autour de **trois axes métier** :

1. **Recrutement** — quels canaux amènent les meilleures recrues (celles qui restent).
2. **Turnover / Attrition** — combien de départs, volontaires ou non, et à quel rythme.
3. **Satisfaction, performance et risque d'attrition** — qui est engagé, qui performe, et qui risque de partir.

## 🗂️ Jeu de données

- **311 lignes** × **36 colonnes** — une ligne par employé (entreprise fictive)
- Source : *Human Resources Data Set* (HRDataset_v14) de Rich Huebner & Dr. Carla Patalano
- Couvre le recrutement (`RecruitmentSource`), le turnover (`DateofHire`, `DateofTermination`, `TermReason`, `EmploymentStatus`), la performance/satisfaction (`PerformanceScore`, `EngagementSurvey`, `EmpSatisfaction`) et l'assiduité (`Absences`, `DaysLateLast30`)

**Limite connue :** le dataset ne contient pas de date d'ouverture de poste, donc un vrai *time-to-fill* ne peut pas être calculé — cette limite a été identifiée et volontairement exclue du périmètre plutôt que d'être forcée artificiellement.

## 🧹 Nettoyage des données (Power Query)

### Colonnes supprimées (19)
| Raison | Colonnes |
|---|---|
| ID numérique redondant avec une colonne texte conservée | `MarriedID`, `MaritalStatusID`, `GenderID`, `EmpStatusID`, `DeptID`, `PerfScoreID`, `PositionID`, `ManagerID` |
| Redondant avec `RecruitmentSource` | `FromDiversityJobFairID` |
| Données géo/personnelles non nécessaires | `State`, `Zip`, `DOB` |
| Attributs personnels sensibles, sans exigence DEI formulée | `Sex`, `MaritalDesc`, `CitizenDesc`, `HispanicLatino`, `RaceDesc` |
| Flag redondant (déjà porté par `EmploymentStatus`) | `Termd` |

### Colonnes conservées (17)
`Employee_Name`, `EmpID`, `Department`, `Position`, `ManagerName`, `Salary`, `DateofHire`, `DateofTermination`, `TermReason`, `EmploymentStatus`, `RecruitmentSource`, `PerformanceScore`, `EngagementSurvey`, `EmpSatisfaction`, `SpecialProjectsCount`, `LastPerformanceReview_Date`, `DaysLateLast30`, `Absences`.

### Points d'attention sur le typage
- Dates au format US (`M/D/YYYY`) → locale forcée en **Anglais (États-Unis)** dans Power Query, pas la locale par défaut.
- `DateofTermination` vide pour les employés actifs — c'est attendu, ne pas combler : c'est ce champ qui distingue actif vs. sorti.
- `Salary` → Nombre décimal fixe, format Devise.
- `EngagementSurvey` → Nombre décimal, 2 décimales (échelle 1–5 déjà correcte).
- `EmpSatisfaction`, `SpecialProjectsCount`, `DaysLateLast30`, `Absences` → Nombre entier.
- `EmpID` → Nombre entier, marqué **Ne pas résumer**.

## 🧩 Modélisation — Schéma en étoile avec dimension à rôles multiples

Contrairement au modèle financier (une ligne par entreprise-trimestre), ce dataset est **au grain employé**, avec **deux dates** rattachées à chaque ligne (embauche + départ). Cela introduit une technique nouvelle : une **dimension à rôles multiples** (*role-playing dimension*) — une seule table de calendrier utilisée deux fois.

**Structure :** `FactEmployee` (1 ligne par employé) → `DimDate` (deux fois) + `DimDepartment`

- `DimDate[Date] → FactEmployee[DateofHire]` — relation **active** par défaut (pilote les "New Hires").
- `DimDate[Date] → FactEmployee[DateofTermination]` — relation **inactive**, activée à la demande dans certaines mesures via `USERELATIONSHIP` (pilote les "Terminations").
- `DimDepartment[DepartmentKey] (1) → FactEmployee[DepartmentKey] (plusieurs)`, avec `Position` imbriquée sous `Department` en hiérarchie.

**Étapes clés :**
1. Charger le CSV, supprimer les 19 colonnes, définir les types.
2. Ajouter la colonne calculée `Tenure (Years)` = durée entre `DateofHire` et (`DateofTermination` ou aujourd'hui) ÷ 365,25.
3. Construire `DimDepartment` par référence de requête : `Department` + `Position`, doublons supprimés, `DepartmentKey` ajouté.
4. Construire `DimDate` **en DAX** (et non Power Query, car elle doit couvrir deux colonnes source) :
   ```dax
   DimDate = CALENDAR(
       MIN(FactEmployee[DateofHire]),
       MAX(TODAY(), MAX(FactEmployee[DateofHire]))
   )
   ```
   Marquer cette table comme table de dates dans la vue Modèle.
5. Dans `FactEmployee`, supprimer `Department`/`Position` (remplacées par `DepartmentKey`), renommer la requête.

## 🧮 Mesures DAX principales

```dax
Active Headcount = CALCULATE(COUNTROWS(FactEmployee), FactEmployee[EmploymentStatus] = "Active")

New Hires = CALCULATE(COUNTROWS(FactEmployee))   -- utilise la relation active DateofHire

Terminations =
CALCULATE(COUNTROWS(FactEmployee), USERELATIONSHIP(DimDate[Date], FactEmployee[DateofTermination]))

Headcount (As of Date) =
CALCULATE(
    COUNTROWS(FactEmployee),
    FILTER(
        ALL(FactEmployee),
        FactEmployee[DateofHire] <= MAX(DimDate[Date]) &&
        (FactEmployee[DateofTermination] > MAX(DimDate[Date]) || ISBLANK(FactEmployee[DateofTermination]))
    )
)

Turnover Rate % = DIVIDE([Terminations], [Headcount (As of Date)])

Voluntary Turnover Rate % =
VAR VolTerms = CALCULATE([Terminations], FactEmployee[EmploymentStatus] = "Voluntarily Terminated")
RETURN DIVIDE(VolTerms, [Headcount (As of Date)])

Avg Engagement Survey = AVERAGE(FactEmployee[EngagementSurvey])
Avg Employee Satisfaction = AVERAGE(FactEmployee[EmpSatisfaction])

% High Performers =
DIVIDE(CALCULATE(COUNTROWS(FactEmployee), FactEmployee[PerformanceScore] = "Exceeds"), [Active Headcount])

Forecasted Hiring Need (Next Qtr) =
VAR ThisQuarterKey = YEAR(TODAY()) * 4 + QUARTER(TODAY())
VAR LastCompleteQuarterKey = ThisQuarterKey - 1
VAR StartQuarterKey = LastCompleteQuarterKey - 2
VAR LastThreeQuarters =
    FILTER(
        VALUES(DimDate[QuarterKey]),
        DimDate[QuarterKey] >= StartQuarterKey && DimDate[QuarterKey] <= LastCompleteQuarterKey
    )
RETURN
AVERAGEX(
    LastThreeQuarters,
    CALCULATE([Terminations], USERELATIONSHIP(DimDate[Date], FactEmployee[DateofTermination]))
)
```
*(nécessite la colonne `QuarterKey = YEAR(DimDate[Date]) * 4 + QUARTER(DimDate[Date])` dans `DimDate`)*

`Forecasted Hiring Need` reprend exactement la logique de moyenne mobile sur 3 trimestres du modèle financier (`Forecast Revenue (3Q Avg)`), appliquée à l'attrition plutôt qu'au revenu, pour répondre à : *combien d'embauches de remplacement prévoir le trimestre prochain ?*

## 📑 Structure du rapport — 2 onglets

### Onglet 1 · Vue d'ensemble Effectifs & Recrutement
Segments (Département / Statut / Année d'embauche / Source), 4 cartes KPI (Effectif actif, Nouvelles embauches, Départs, Taux de turnover), graphique en cascade "Pont d'effectif" (début → +embauches → −départs → fin), graphique combiné embauches vs. départs avec **prévision native Power BI** sur l'effectif net, treemap d'efficacité des canaux de recrutement (taille = embauches, couleur = taux de rétention), table du personnel exportable avec barres de données.

### Onglet 2 · Performance, Satisfaction & Risque d'attrition
Segments (Département / Manager / Performance), 4 cartes KPI (Engagement moyen, Satisfaction moyenne, % Hauts performeurs, Taux de turnover volontaire), nuage de points Satisfaction vs. Engagement (bulle = ancienneté, couleur = département), jauge Satisfaction vs. objectif de référence, visuel **Influenceurs clés** (IA) pour identifier les facteurs qui augmentent le risque de départ, **Arborescence de décomposition** (IA) pour explorer où se concentre l'attrition (Département → Source → Performance).

## 🛠️ Compétences démontrées

- Nettoyage de données avec gestion de locale (dates US) et de colonnes sensibles/PII
- Modélisation d'une **dimension à rôles multiples** (`USERELATIONSHIP`)
- Construction d'une table de dates en DAX (`CALENDAR`) plutôt qu'en Power Query
- Mesures DAX avancées : effectif à une date donnée, taux de turnover, prévision par moyenne mobile
- Utilisation des **visuels IA natifs** de Power BI (Influenceurs clés, Arborescence de décomposition)
- Conception d'un rapport RH bilingue (modèle en anglais, interface en français)

## 🚀 Utilisation

1. Ouvrir le fichier `.pbix` dans Power BI Desktop.
2. Rafraîchir les données si nécessaire (Accueil → Actualiser).
3. Naviguer entre les deux onglets via la barre de navigation en bas du rapport.

## 📄 Licence

Projet réalisé à des fins de démonstration / portfolio.
