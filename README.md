# Reporting Portfolio

Outil de reporting de portefeuille de projets — suivi budgétaire, respect des jalons et identification des risques, en un seul fichier HTML autonome, sans installation ni serveur.

## À propos

**Reporting Portfolio** consolide les extractions Excel mensuelles d'un outil de pilotage projet pour restituer une vue synthétique du portefeuille. Il couvre l'avancement budgétaire, le suivi des jalons de delivery et la détection des projets ou données à risque.

Conçu pour fonctionner entièrement **côté client** : aucune donnée ne quitte le poste, aucune infrastructure n'est nécessaire.

## Écrans

| Écran | Contenu |
|---|---|
| **Synthèse portfolio** | Vue d'ensemble par projet : météos, tendances, taux de consommation, évolution vs période précédente |
| **Synthèse budgétaire** | Budget de référence, avancement calendaire, indicateur conforme / sur-conso. / sous-conso. |
| **Jalons Projets** | KPIs de delivery, segments de risque (en retard, à risque imminent, ≤30/60/90j), filtres rapides multi-valeurs |
| **Risk Management** | Projets en difficulté, cohérence des météos, risques masqués, contrôles de cohérence des données |

S'ajoutent un panneau **Paramétrage** (filtres d'entrée, période d'analyse des jalons) et une **Aide** intégrée, accessibles via les icônes ⚙ et ? de la barre d'application.

## Fonctionnalités clés

- **Filtres d'entrée configurables** — par intitulé projet (correspondance `%valeur%`) et par statut de suivi ; appliqués dynamiquement sur tous les écrans sans réimport.
- **Comparaison N vs N-1** — marqueur visuel sur les projets ayant évolué entre deux périodes ; bouton « Évolutions N-1 » pour filtrer sur ces seuls projets et afficher la météo précédente.
- **Période d'analyse des jalons** — filtre de dates (début/fin) appliqué transversalement à tous les calculs jalons.
- **Filtres rapides jalons** — par projet, statut, direction, chef de projet ; multi-valeurs, cumulatifs, sans persistance entre sessions.
- **Risques masqués** — détection des projets à météo globale verte portant des jalons en retard.
- **7 contrôles de cohérence projet** + **7 contrôles de cohérence jalons** — détection des anomalies de qualité de données.

## Architecture

- **Fichier HTML unique et autonome** — aucune installation, aucun serveur, aucune dépendance externe.
- Fonctionne **hors ligne**, ouvert directement dans Chrome (`file://`).
- Import `.xlsx` et traitements (filtres, calculs, météos, jalons) **entièrement côté client**.
- **Aucune donnée ne quitte le poste**.
- Persistance locale automatique (IndexedDB) + sauvegarde/restauration JSON (`DB.json`) pour l'historique multi-périodes.
- **Aucune donnée d'exemple embarquée** : l'outil s'ouvre vide au premier lancement.

## Démarrage rapide

1. Télécharger `Reporting_Portfolio.html` et l'ouvrir dans **Chrome**.
2. Cliquer sur **Importer Synthèse Projets**, sélectionner le fichier `.xlsx` et la période (mois / année).
3. *(Optionnel)* Cliquer sur **Importer les jalons** pour activer l'écran Jalons Projets et ses contrôles de cohérence.
4. Configurer les filtres d'entrée via l'icône **⚙ Paramétrage** si besoin.
5. Réimporter chaque mois pour alimenter la comparaison N vs N-1.

## Contenu du dépôt

| Fichier | Description |
|---|---|
| `Reporting_Portfolio.html` | L'outil — fichier unique à télécharger et ouvrir dans Chrome. |
| `Reporting_Portfolio_Specification.md` | Spécification fonctionnelle complète : règles de gestion, calculs, écrans, architecture. |
| `Test_files.zip` | Jeux de données fictives pour tester les règles de gestion (voir détail ci-dessous). |

### Détail de `Test_files.zip`

| Fichier | Rôle |
|---|---|
| `Extraction_ATS_test_octobre_2026.xlsx` | Extraction projet — période courante. 21 projets couvrant les règles de filtrage, de cohérence météo et de contrôle budgétaire. |
| `Extraction_ATS_test_septembre_2026.xlsx` | Extraction projet — période N-1, pour tester la comparaison et le marqueur d'évolution. |
| `Extraction_Jalons_test_octobre_2026.xlsx` | Extraction jalons — période courante. 25 jalons couvrant les segments de risque et les 7 contrôles de cohérence. |
| `Extraction_Jalons_test_septembre_2026.xlsx` | Extraction jalons — période N-1, allégée, pour tester la bascule de période. |

**Ordre d'import recommandé :** projet septembre → projet octobre → jalons octobre.
La spécification détaille les règles testées par chaque ligne du jeu de données.

## Statut

Version actuelle : **v0.15**. Spécification et outil évoluent conjointement — toute évolution fonctionnelle est d'abord consignée dans la spécification avant implémentation.
