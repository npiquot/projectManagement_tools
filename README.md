# Reporting Portfolio

Outil de reporting de portefeuille de projets — suivi budgétaire, respect des jalons et identification des risques, en un seul fichier HTML autonome.

## À propos

**Reporting Portfolio** consolide les extractions Excel mensuelles de l'outil de pilotage projet pour restituer une vue synthétique du portefeuille : avancement budgétaire au regard du calendaire, respect des jalons de delivery, et identification des projets et données à risque. 

## Écrans

| Écran | Contenu |
|---|---|
| **Synthèse portfolio** | Vue d'ensemble par projet : météos, tendances, taux de consommation, évolution vs N-1 |
| **Synthèse budgétaire** | Budget de référence, avancement calendaire, indicateur conforme / sur-conso. / sous-conso. |
| **Jalons Projets** | KPIs de delivery, segments de risque (en retard, à risque imminent, échéances ≤30/60/90j), cartographie (répartition par statut, concentration, risques masqués) |
| **Risk Management** | Projets en difficulté, cohérence des météos, contrôles de cohérence des données (projets et jalons) |

S'ajoutent un panneau **Paramétrage** (filtres d'entrée configurables) et une **Aide** intégrée, accessibles hors navigation principale.

## Architecture & exécution

- **Fichier HTML unique et autonome** — aucune installation, aucun serveur.
- Fonctionne **hors ligne**, ouvert directement dans Chrome (`file://`).
- Import des extractions `.xlsx` et tout le traitement (filtres, calculs, météos) **réalisés intégralement côté client**.
- **Aucune donnée ne quitte le poste** : pas de backend, pas d'upload.
- Persistance locale automatique (IndexedDB) + export/import JSON pour l'historique multi-périodes.
- Le code source ne contient **aucune donnée d'exemple** : l'outil démarre vide à chaque premier lancement.

## Démarrage rapide

1. Télécharger `Reporting_Portfolio.html` et l'ouvrir dans Chrome.
2. Cliquer sur **Importer Synthèse Projets**, sélectionner le `.xlsx` mensuel et la période concernée.
3. (Optionnel) Cliquer sur **Importer les jalons** pour activer l'écran Jalons Projets et ses contrôles de cohérence dédiés.
4. Affiner les filtres d'entrée via l'icône **⚙ Paramétrage** si besoin.

## Contenu du dépôt

| Fichier | Description |
|---|---|
| `Reporting_Portfolio.html` | L'outil — fichier unique à télécharger et ouvrir dans Chrome. |
| `Reporting_ATS_Specification_v1.md` | Spécification fonctionnelle complète (source de données, règles de calcul, écrans, architecture technique). |
| `Test_files.zip` | Jeux de données d'exemple pour tester les règles de gestion (voir détail ci-dessous). |

### Détail de `Test_files.zip`

| Fichier | Rôle |
|---|---|
| `Extraction_ATS_test_octobre_2026.xlsx` | Extraction projet — période courante. 22 projets couvrant l'ensemble des règles de filtrage, de cohérence météo et de contrôle budgétaire. |
| `Extraction_ATS_test_septembre_2026.xlsx` | Extraction projet — période N-1, pour tester la comparaison et le marqueur d'évolution. |
| `Extraction_Jalons_test_octobre_2026.xlsx` | Extraction jalons — période courante. 25 jalons couvrant les segments de risque et les 7 contrôles de cohérence dédiés. |
| `Extraction_Jalons_test_septembre_2026.xlsx` | Extraction jalons — période N-1, allégée, pour tester la bascule de période. |

*Ordre d'import recommandé : projet septembre → projet octobre → jalons octobre (cf. spécification pour le détail des règles testées par chaque ligne).*

## Statut

Version actuelle : **v0.13**. Spécification et outil évoluent conjointement — toute évolution fonctionnelle est d'abord consignée dans `Reporting_ATS_Specification_v1.md` avant implémentation.

---
