# Reporting ATS — Spécification fonctionnelle
**Version :** 0.12 (en cours de spécification)
**Statut :** Brouillon soumis à validation — *développement non démarré*
**Propriétaire :** Direction Architecture & Standards Technologique (ATS)

---

## 1. Contexte & objectif

L'outil **« Reporting ATS »** a pour but de permettre au directeur du département Architecture & Standards Technologique de **suivre l'avancement de son portefeuille de projets** et de chacun des projets pris individuellement.

Les chefs de projet renseignent des fiches de synthèse dans l'outil projet de la DSI. Une **extraction Excel mensuelle** de cet outil constitue la source de données de Reporting ATS. L'outil est donc **découplé** de l'outil projet : pas d'intégration temps réel, alimentation par import au rythme mensuel.

---

## 2. Source de données

### 2.1 Format du fichier
- Fichier **Excel** comportant un onglet nommé **`Export`**.
- **Granularité : une ligne par projet.**
- **Clé unique et stable d'un mois sur l'autre : `ID Initiative`.**

### 2.2 Dictionnaire des colonnes

| Catégorie | Colonnes |
|-----------|----------|
| **Identification & cadrage** | ID Groupe · ID Initiative · Intitulé projet · Chef de projet · Direction · Client/Pôle · Objectifs/Enjeux · Status |
| **Budget — Engagé** | Budget engagé JH · Budget engagé € |
| **Budget — Initial** | Budget initial JH · Budget initial € |
| **Budget — Révisé** | Budget révisé JH · Budget révisé € |
| **Budget — Consommé** | Budget consommé JH · Budget consommé € |
| **Budget — Facturé** | Facturé JH · Facturé € |
| **Suivi** | Suivi dans Pulse · Date de saisie |
| **Santé — Météos (5 axes)** | Météo globale · Météo budget · Météo planning · Météo périmètre · Météo sponsor |
| **Santé — Tendances (5 axes)** | Tendance globale · Tendance budget · Tendance planning · Tendance périmètre · Tendance sponsor |
| **Narratif** | Principales réalisations · Risques et plan de mitigation · Prochaines étapes |

### 2.3 Référentiels de valeurs

- **`ID Groupe`** : identifiant du portfolio groupe — **hors périmètre** de notre cas d'usage (conservé comme attribut informatif, ne structure pas le suivi).
- **`Status`** (5 valeurs) : `Non démarré` · `En murissement` · `En réalisation` · `Terminé` · `Abandonné`.
- **Météos** (3 valeurs, en anglais) : `green` = au vert · `yellow` = point à surveiller · `red` = difficulté majeure.
- **Tendances** (3 valeurs) : `Good` = amélioration · `Neutral` = maintien · `Bad` = dégradation.

---

## 3. Import des extractions & mécanisme d'historique

### 3.1 Import d'une extraction
- Bouton explicite **« Importer l'extraction »** → sélection du fichier `.xlsx` issu de l'outil de pilotage projet BPCE (données actualisées du mois).
- Au moment de l'import, l'utilisateur **saisit la période** via **deux listes déroulantes** :
  - **Mois** : janvier → décembre,
  - **Année** : **2026 à 2030**.
- La période (mois + année) ainsi saisie **identifie l'extraction**. À l'import, **les lignes brutes sont conservées** et rattachées à cette période ; **les filtres (§4) ne sont pas figés à l'import** mais **appliqués dynamiquement à l'affichage** (ils sont paramétrables, cf. §4 et §12).
- **Réimport d'une période déjà présente** : l'extraction de cette période est **mise à jour / remplacée** (pas de doublon).

> À distinguer du `DB.json` (§8.3) : l'import `.xlsx` fait **entrer la donnée source** ; le `DB.json` **conserve l'état** de l'outil (historique des extractions déjà importées).

### 3.2 Historique & comparaison
- Alimentation par **imports successifs**, période après période.
- L'outil **compare deux périodes consécutives** (N vs N-1) pour produire les indicateurs d'évolution.
- **Périmètre V1 : démarrage avec deux périodes** (N et N-1).
- **Architecture cible :** conçue pour accepter des imports successifs au-delà de deux, sans refonte ultérieure.

### 3.3 Import complémentaire — « Export Pulse Jalon »
- Un **second import**, en complément de l'« Export Pulse », alimente l'écran **Jalons Projets** (cf. §11).
- Fichier `.xlsx` ; l'outil lit **le premier onglet**, quel que soit son nom.
- Sélection **mois + année** identiques à l'import projet ; le couple identifie l'extraction jalon ; ré-import du même mois = remplacement.
- Croisement avec les projets **uniquement pour une même période** (cf. §11.1).

---

## 4. Filtres d'entrée (configurables)

Les filtres d'entrée déterminent les projets pris en compte. Ils sont **paramétrables via l'écran « Paramétrage »** (§12) et **appliqués dynamiquement à l'affichage** (cf. §3.1). **Tous les filtres sont insensibles à la casse.**

**1. Intitulé projet** — liste de valeurs configurables. Un projet satisfait ce critère si son `Intitulé projet` **contient** au moins une des valeurs, **quelle que soit la position** dans l'intitulé (correspondance de type `%valeur%`, logique **OU**).
- Valeurs **par défaut** : « AEO », « PIAT ».
- **Liste vide → critère non appliqué** : tous les projets passent ce critère.

**2. Suivi dans Pulse** — mode configurable :
- **« Oui »** (défaut) : le projet doit avoir `Suivi dans Pulse` = « Oui ».
- **« ALL »** : ce filtre n'est pas appliqué.

**Règle combinée :** un projet est retenu si **(critère Intitulé satisfait)** **ET** **(`Suivi dans Pulse` = « Oui » OU mode « ALL »)**.

> **Règle générale (transverse) :** sauf spécification explicite contraire, ces filtres s'appliquent à **tous les écrans** de l'outil (y compris la restriction des jalons aux projets retenus, §11.1). Toute modification dans l'écran « Paramétrage » **recalcule immédiatement l'ensemble des écrans**, sans réimport.

---

## 5. Écran 1 — « Synthèse portfolio »

Vue tabulaire : **une ligne par projet** filtré.

### 5.1 Colonnes affichées

| # | Colonne | Règle d'affichage |
|---|---------|-------------------|
| 1 | Intitulé projet | tel quel |
| 2 | Chef de projet | tel quel |
| 3 | Objectifs/Enjeux | **100 premiers caractères** uniquement |
| 4 | Statut | tel quel |
| 5 | Budget de référence JH | **révisé** si renseigné, **sinon initial** |
| 6 | Budget de référence € | révisé si renseigné, sinon initial — **en k€** |
| 7 | Budget consommé JH | tel quel |
| 8 | Budget consommé € | **en k€** |
| 9 | Taux de consommation € | **calculé, en %** (voir §5.4) |

### 5.2 Météos
- **Météo globale en majeur** (élément dominant de chaque ligne).
- En plus petit : **budget · planning · périmètre**.
- **Météo sponsor : exclue** — *règle générale, l'axe sponsor n'est affiché nulle part dans l'outil.*
- Représentation en **icônes** : ☀️ soleil = `green` · ☁️ nuage = `yellow` · ⛈️ orage = `red`.

### 5.3 Tendances
- Représentation en **flèches** : ↗️ = `Good` (amélioration) · ➡️ = `Neutral` (maintien) · ↘️ = `Bad` (dégradation).

### 5.4 Règles de calcul

**Budget de référence** (colonnes 5 et 6, JH et €) :
> Budget révisé s'il est renseigné dans le fichier, sinon budget initial.
> *Aucun repère visuel ne distingue un budget révisé d'un budget initial : affichage sobre de la valeur de référence.*

**Conversion en k€** : les montants en euros des colonnes 6 et 8 sont exprimés en **milliers d'euros (k€)**.

**Taux de consommation € (%)** :
> `Taux = Budget consommé € / Budget de référence €`
> où le **budget de référence = révisé € s'il est renseigné, sinon initial €**.
> Le dénominateur du taux est donc **identique** au budget affiché en colonne 6 (cohérence de lecture garantie).

### 5.5 Marqueur d'évolution vs mois précédent
- Met en évidence visuellement tout **changement de météo OU de tendance** entre les deux derniers imports.
- **Périmètre : les 4 axes affichés** (globale, budget, planning, périmètre).
- Conséquence : un projet dont *un seul* des 4 axes bouge (ex. budget passant au rouge) est signalé comme « a évolué ».

---

## 6. Écran 2 — « Synthèse budgétaire »

Vue tabulaire regroupant tous les projets du département, sur la **base des mêmes filtres que l'écran 1** (§4).

### 6.1 Bascule d'unité
- En haut du tableau, sélecteur d'**unité d'affichage** : **k€** ou **JH**.
- S'applique aux colonnes monétaires/charge (*Consommé à date* et *Budget*).

### 6.2 Colonnes affichées

| # | Colonne | Règle |
|---|---------|-------|
| 1 | Nom du projet | tel quel |
| 2 | Consommé à date | `Budget consommé` (€ ou JH selon la bascule §6.1) |
| 3 | Budget | révisé si renseigné, sinon initial (même logique qu'écran 1) — unité selon §6.1 |
| 4 | Taux d'avancement | `consommé / budget`, en % *(équivalent au taux de consommation de l'écran 1)* |
| 5 | Indicateur visuel | conforme / sous-consommation / sur-consommation (voir §6.4) |

### 6.3 Avancement calendaire — calendrier de capacité

L'année de référence est **alignée sur le calendrier civil (1ᵉʳ janvier → 31 décembre)**, mais la **capacité de travail disponible n'est pas uniforme**. Le mois est choisi à l'import ; l'avancement est arrêté à la **fin du mois importé (mois inclus)**, car la consommation de l'extraction est cumulée à la **fin de ce mois** (la colonne `Date de saisie` n'étant pas considérée fiable). Importer **juin** positionne donc l'avancement à **fin juin**.

**Pour chaque mois, calcul de la `capacité_mois` :**

1. `jours_ouvrés = jours de semaine (lun–ven) − jours fériés français du mois`
2. Si **juillet** ou **août** → multiplier par **0,6** (taux de présence 60 %)
3. Si **mai**, **février** ou **décembre** → **soustraire 5 jours** (forfait, *en sus* des jours fériés déjà retirés)

> **Ordre des opérations** pour juillet/août : retirer d'abord les jours fériés, **puis** appliquer le facteur 0,6 sur les jours restants.

**Agrégats :**
- `capacité_annuelle` = somme des `capacité_mois` sur les 12 mois.
- `capacité_écoulée` = somme des `capacité_mois` de **janvier jusqu'au mois importé inclus**.
- **`Avancement calendaire (%) = capacité_écoulée ÷ capacité_annuelle`**.
- Conséquence : **décembre = 100 %** (année complète).

> **Hypothèse de référence :** le **budget de référence est une enveloppe annuelle**. L'alignement consommé / avancement calendaire est donc pertinent à l'échelle de l'année civile.

### 6.4 Indicateur visuel — règle de bascule

Écart = `taux d'avancement − avancement calendaire`, exprimé en **points de pourcentage**. Tolérance de **±10 points** :

| Condition | État |
|-----------|------|
| `|écart| ≤ 10` | ✅ **Conforme** |
| `taux < avancement calendaire − 10` | 🔵 **Sous-consommation** |
| `taux > avancement calendaire + 10` | 🔴 **Sur-consommation** |

### 6.5 Ligne de total
- Colonnes *Consommé à date* et *Budget* : **somme** de toutes les lignes.
- *Taux d'avancement* du total : **recalculé** sur les agrégats → `total consommé ÷ total budget`.
- *Indicateur* du total : appliqué à ce taux global, comparé au même avancement calendaire (§6.3–6.4).

### 6.6 Observation de conception
Le budget de référence étant une **enveloppe annuelle** (cf. §6.3), l'hypothèse de **consommation linéaire sur l'année civile** est cohérente et l'indicateur est pertinent. Réserve résiduelle : un projet **actif sur une partie seulement de l'année** (démarrage ou clôture en cours d'exercice) apparaîtra mécaniquement en sous- ou sur-consommation au regard de l'avancement calendaire global. Un calage sur les dates propres à chaque projet nécessiterait des colonnes de dates absentes de l'extraction actuelle. *(En réserve — cf. §10.)*

---

## 7. Écran 3 — « Risk Management »

**Objectif** : faire ressortir les projets en difficulté, pour un pilotage par l'alerte. Filtres de base appliqués (§4).

### 7.1 Population & ordre
- L'écran n'affiche **que les projets dont la météo globale est `yellow` ou `red`** ; les `green` sont **exclus**.
- **Axe déclencheur : uniquement la météo globale** (les axes secondaires ne conditionnent pas l'inclusion).
- Tri par **sévérité : `red` d'abord, puis `yellow`**.

### 7.2 Indicateur de début de ligne — alerte mitigation
- En tête de chaque ligne, un indicateur signale les projets de cette liste dont la colonne **`Risques et plan de mitigation` est vide** (vide ou ne contenant que des espaces).
- Sémantique : *« projet en difficulté sans plan de mitigation documenté »* — point d'attention de gouvernance prioritaire (un projet à risque sans mitigation renseignée est précisément celui sur lequel alerter).

### 7.3 Contenu par projet
- **Identification** : `ID Initiative`, intitulé projet, chef de projet (+ statut et direction pour le contexte).
- **Santé (contexte)** : **uniquement la météo globale et sa tendance** — les météos secondaires (budget / planning / périmètre) ne sont **pas** reprises sur cet écran.
- **Éléments narratifs**, dans cet ordre : Principales réalisations → Risques et plan de mitigation → Prochaines étapes.

### 7.4 Affichage des textes narratifs
- Présentation **en cartes** (une carte par projet), plus lisible que le tableau pour du texte.
- Narratif **tronqué** par défaut.
- **Détail complet** révélé au choix :
  - **au survol** de la souris, ou
  - **au clic** — le détail ouvert se referme lorsqu'on clique en dehors.

### 7.5 Seconde zone — Contrôle de cohérence des météos

**Objectif** : détecter les incohérences entre la **météo globale** et les **météos spécifiques** (globale trop optimiste ou trop pessimiste au regard des axes).

**Périmètre d'analyse** : **tous** les projets passant les filtres de base (§4), *indépendamment* de la restriction jaune/rouge de la première zone — une incohérence peut concerner un projet à globale **verte**. Seuls les projets **en anomalie** sont listés.

**Axes spécifiques considérés** : budget, planning, périmètre. **Sponsor exclu.**

**Règle** (tolérance d'un seul jaune) :

| Globale | Axes spécifiques (budget · planning · périmètre) | Verdict | Type d'anomalie |
|---------|---------------------------------------------------|---------|-----------------|
| verte | 0 rouge **et** ≤ 1 jaune | cohérent | — |
| verte | ≥ 2 jaunes (0 rouge) | **anomalie** | globale sous-évaluée |
| verte | ≥ 1 rouge | **anomalie** | globale sous-évaluée |
| jaune | tous verts | **anomalie** | globale sur-évaluée |
| jaune | au moins un jaune ou rouge | cohérent | — |
| rouge | ≥ 1 rouge | cohérent | — |
| rouge | 0 rouge | **anomalie** | globale sur-évaluée |

**Types d'anomalie** :
- *Globale sous-évaluée* : la globale est plus optimiste que les axes — elle masque un risque.
- *Globale sur-évaluée* : la globale est plus pessimiste que les axes — alarme non justifiée.

**Affichage** :
- Liste des projets en anomalie, affichant la **météo globale ET les 3 météos spécifiques** (exception locale au §7.3, pour rendre l'incohérence lisible).
- L'anomalie est signalée par une **icône dédiée** (jeu « désaccord météo », sans libellé textuel à l'écran) :
  - *Globale sous-évaluée* → **soleil (vert) marqué d'une pastille d'alerte rouge** — façade calme masquant un risque réel ;
  - *Globale sur-évaluée* → **orage (rouge) marqué d'une pastille de validation verte** — alarme démentie par les axes.
- Le libellé qualifiant (« globale sous-évaluée » / « globale sur-évaluée ») est fourni : en **infobulle au survol** de l'icône (attribut `title`) et en **texte de remplacement** si l'image est absente (`alt` pour une image, ou nom accessible équivalent `aria-label` / `<title>` pour une icône SVG inline).

### 7.6 Troisième zone — Contrôles de cohérence des données

**Objectif** : détecter les anomalies de **qualité / cohérence des fiches**, au-delà de la cohérence météo (§7.5).

**Périmètre & affichage** : tous les projets passant les filtres de base (§4). Seuls les projets présentant **au moins une anomalie** sont listés, **une ligne par projet** ; les anomalies sont affichées en **libellés courts** (un projet peut en cumuler plusieurs). Toutes les analyses budgétaires se font **en euros** ; le **budget de référence** est le révisé s'il est renseigné, sinon l'initial.

**Contrôles :**

| # | Libellé | Règle | Périmètre / pré-requis |
|---|---------|-------|------------------------|
| 1 | Budget nul, consommation présente | budget de référence € = 0 (ou vide) **et** consommé € > 0 | — |
| 2 | Non démarré mais consommé | statut `Non démarré` **et** consommé € > 0 | — |
| 3 | Météo globale absente | météo globale non renseignée | **hors** statut `Non démarré` |
| 4 | Dépassement d'enveloppe | consommé € > budget de référence € | — |
| 5 | Identification incomplète | chef de projet **ou** direction non renseigné | — |
| 6 | Narratif incomplet sur projet à risque | météo globale **rouge ou jaune** **et** (« Principales réalisations » **ou** « Prochaines étapes » vide) | — |
| 7 | Atterrissage projeté hors cible | voir §7.6.1 | statuts **actifs** ; budget de référence € > 0 |

#### 7.6.1 Contrôle de projection (atterrissage)

Extrapolation de la consommation actuelle à l'année pleine via l'avancement calendaire (§6.3, tel qu'utilisé à l'écran 2) :

> `atterrissage_projeté € = consommé € ÷ (avancement_calendaire / 100)`

Comparaison au budget de référence € avec une **tolérance de ±5 %** :
- `atterrissage_projeté > budget de référence € × 1,05` → **dépassement projeté**
- `atterrissage_projeté < budget de référence € × 0,95` → **sous-réalisation projetée**
- sinon → atterrissage conforme (pas d'anomalie)

**Périmètre du contrôle** : projets **actifs uniquement** (`En réalisation`, `En murissement`), avec **budget de référence € > 0**.

#### 7.6.2 Contrôles de cohérence des données — jalons

Affichés **uniquement si un fichier jalon est importé pour la période courante** (cf. §11.1). Un **sous-tableau** dédié liste, **une ligne par jalon** présentant au moins une anomalie (colonnes : Projet · Jalon · étiquettes d'anomalie). Contrôles :

1. Statut « Réalisé » ou « Anticipé » mais **% ≠ 100**.
2. **% = 100** mais statut **∉** {Réalisé, Anticipé}.
3. Statut « Non commencé » mais **% > 0**.
4. Statut « En retard » mais **% = 100**.
5. **Jalon non exploitable** : aucune des quatre échéances renseignée.
6. **Échéance finale antérieure à l'échéance initiale**.
7. **Statut absent** (champ vide).

---

## 8. Exigences techniques & architecture

### 8.1 Contexte d'exécution
- Outil exécuté sur le **poste de travail professionnel** de l'utilisateur.
- **Aucune installation de logiciel** possible sur ce poste.
- Navigateur cible : **Google Chrome** (accès internet disponible mais **non requis** au fonctionnement).
- Java présent sur le poste mais **non utilisé** dans l'architecture retenue.

### 8.2 Architecture retenue — application web 100 % côté client
- **Un unique fichier HTML autonome**, embarquant (inlinées) la structure, le style, la logique JavaScript et les bibliothèques nécessaires (parsing `.xlsx` type SheetJS). **Fonctionne hors ligne.**
- Ouverture directe dans Chrome (`file://`), sans serveur.
- L'extraction Excel est chargée via un champ d'import de fichier, **parsée et traitée intégralement côté client** (filtres, météos, tendances, calendrier de capacité, taux, totaux).
- **Aucune donnée ne quitte le poste** : pas de backend, pas d'upload, pas d'appel sortant. Atout déterminant en contexte bancaire (confidentialité, DORA).

### 8.3 Persistance des données
La continuité automatique et la persistance fichier sont **dissociées** :

- **Continuité automatique (IndexedDB)** : l'application mémorise le dernier état dans le stockage local du navigateur et le **recharge au démarrage**, sans action de l'utilisateur. Fiable sous Chrome, y compris en `file://`.
- **Persistance fichier (JSON)** — contenant l'**historique de tous les imports** (support du mécanisme N vs N-1 et au-delà) :
  - Bouton **Sauvegarder** (présent sur **tous les écrans**) → télécharge `DB.json`.
  - Bouton **Historiser** → télécharge un fichier horodaté `DB_AAAAMMJJ_HHMMSS.json` (évite les doublons, permet le versionnement).
  - Bouton **Ouvrir un historique** → sélecteur de fichier pour recharger un JSON antérieur (retour arrière).
- **Repli** si le stockage navigateur s'avère instable sur le poste : chargement manuel de `DB.json` au démarrage (un clic). L'outil reste pleinement fonctionnel.

### 8.4 Limite assumée
En raison du bac à sable du navigateur, une page web **ne peut pas** créer/écraser ni relire silencieusement un fichier à un **chemin fixe** (ex. un `DB.json` placé automatiquement à côté du HTML). L'enregistrement passe donc par un téléchargement, et le rechargement par un sélecteur de fichier (ou par IndexedDB pour la continuité auto). L'API *File System Access* (qui s'en rapprocherait) est **indisponible en `file://`** et nécessiterait un contexte `localhost` (écarté avec Java).

### 8.5 Livrable
**Un seul fichier HTML**, conservé sur le poste et ouvert hors ligne dans Chrome.

---

## 9. Style visuel & charte graphique (validé)

Style retenu : **identité du Groupe BPCE**. Registre institutionnel et corporate, sobre, adapté à une diffusion en DSI bancaire.

### 9.1 Palette
| Rôle | Couleur | Hex |
|------|---------|-----|
| Violet profond (barre d'application, titres, valeurs) | aubergine BPCE | `#300B3F` |
| Mauve (accents, marqueur d'évolution, éléments actifs) | purple BPCE | `#714A80` |
| Gris ardoise (texte secondaire) | comet | `#515772` |
| Fonds | blanc / gris violacé clair | `#FFFFFF` / `#F4F1F6` |

Statuts sémantiques (météo) : vert `#2F9E6B`, jaune/ambre `#E0A21A`, rouge `#D6453F`.

### 9.2 Structure d'écran
- **Barre d'application** en violet profond (`#300B3F`) : nom de l'outil « Reporting ATS » à gauche ; à droite, action principale **Importer l'extraction** (ouvre la sélection du `.xlsx` + les listes déroulantes mois/année — cf. §3.1), puis **Sauvegarder · Historiser · Ouvrir un historique** (présents sur tous les écrans — cf. §8.3).
- **Bandeau de titre** : titre de l'écran + sous-titre, et **chips de filtres** (AEO, PIAT, Suivi Pulse : Oui, sélecteur de mois).
- **Tableau** : une ligne par projet, séparateurs fins, fond clair.

### 9.3 Conventions iconographiques
- **Météo** (icône) : ☀ soleil = `green` · ☁ nuage = `yellow` · ⛈ orage = `red`. La **météo globale** est l'icône majeure de la ligne.
- **Météos secondaires** : 3 pastilles colorées dans l'ordre fixe **budget · planning · périmètre** (sponsor exclu — cf. §5.2). *Non reprises sur l'écran Risk Management (cf. §7.3), sauf dans la zone de contrôle de cohérence (§7.5).*
- **Tendance** : flèche ↗ `Good` · → `Neutral` · ↘ `Bad`.
- **Marqueur d'évolution** : pastille d'accent mauve (`#714A80`) signalant un projet dont un des 4 axes a changé (météo ou tendance) depuis le mois précédent (cf. §5.5).
- **Budget** : valeurs en k€, avec un **mini-bargraph** du taux de consommation (coloré selon le niveau).
- **Anomalies de cohérence** (écran Risk Management, §7.5) : jeu « désaccord météo » — *soleil + pastille rouge* = globale sous-évaluée ; *orage + pastille verte* = globale sur-évaluée. Libellé en infobulle (`title`) et en texte de remplacement (`alt` / nom accessible).

### 9.4 Référence
Maquette de l'écran 1 « Synthèse portfolio » validée dans ce style (rendu SVG).

---

## 10. Points en réserve (à arbitrer ultérieurement)

- Affichage éventuel d'une indication lorsque le budget de référence utilisé est le révisé (actuellement non retenu).
- Suivi de tendances sur plus de deux mois une fois l'historique multi-fichiers en place.
- Calage de l'avancement calendaire sur les dates de début/fin propres à chaque projet (nécessite des colonnes de dates non présentes dans l'extraction actuelle).

---

## 11. Écran 4 — « Jalons Projets »

**Objectif :** suivre le delivery des projets, sécuriser le respect des jalons et identifier les zones de risque. Un **4ᵉ onglet** « Jalons Projets » est ajouté à la barre de navigation.

### 11.1 Source, import et appariement par période
- Fichier « Export Pulse Jalon » (cf. §3.3) ; **premier onglet** lu.
- **Colonnes attendues** : `ID initiative`, `Intitulé initiative`, `jalon`, `Échéance Initiale`, `Échéance actualisée IT`, `Echéance révisée Métier`, `Échéance finale`, `% Avancement`, `Statut`, `Cause de retard`, `Détails`, `Client / pôle`, `Client / Direction`, `DSI`, `Chef de projet`, `Jalon affiché dans Pulse`.
- **Clé de jointure** : `ID Initiative` vers l'« Extraction Projet ».
- **Filtre** : on conserve **toutes les lignes**, restreintes aux **jalons des projets retenus** par les filtres §4 (AEO/PIAT + Suivi dans Pulse = Oui). **Pas** de filtre sur « Jalon affiché dans Pulse ».
- **Appariement par période** : les jalons ne sont croisés avec les projets que pour une **même période**. L'écran affiche les jalons de la **période projet courante** ; si aucune extraction jalon n'existe pour ce mois, l'écran affiche un **état vide** (invitation à importer le fichier jalon du mois). Les imports jalon sont **conservés par période** (comme les imports projet).
- **Dépendances** : l'écran nécessite l'import **projet** (jointure, filtre, météo) **et** l'import **jalon** de la même période ; à défaut, état vide.

### 11.2 Définitions
- **Échéance de référence** : 1ʳᵉ renseignée parmi finale → révisée Métier → actualisée IT → initiale.
- **Date d'observation** : fin du mois importé (cf. §6.3).
- **% Avancement** ∈ {0, 25, 50, 75, 100}.
- **Statut jalon** ∈ {Abandonné, Anticipé, En cours, En retard, Non commencé, Réalisé, Revu, (vide)}.
- **Soldé** = statut ∈ {Réalisé, Anticipé} **ou** % = 100. **Abandonné** = statut Abandonné. **Actif** = ni soldé ni abandonné.
- **En retard** = jalon actif **et** (statut « En retard » **ou** échéance de référence < date d'observation).
- **À venir** = jalon actif, non en retard, échéance de référence ≥ date d'observation.
- **Dérive (jours)** = échéance de référence − échéance initiale (> 0 = glissement).
- **Replanifié** = une échéance (IT, Métier ou finale) diffère de l'initiale, **ou** statut « Revu ».
- **Écart IT / Métier** = échéances actualisée IT et révisée Métier renseignées **et** différentes.

### 11.3 Bandeau d'indicateurs (KPIs) — axe « respect des échéances »
- **Taux de jalons en retard** = jalons en retard ÷ **jalons actifs** (et valeur absolue).
- **Dérive moyenne (jours)** = moyenne de la dérive sur les jalons ayant glissé (dérive > 0).
- **Taux de replanification** = jalons replanifiés ÷ total.
- **Écart IT / Métier** = nombre de jalons en divergence IT vs Métier.
- **À livrer ≤ 30 j** = nombre de jalons à venir dont l'échéance de référence ≤ date d'observation + 30 jours.

### 11.4 Tableau des jalons — axe de vue sélectionnable
- **Colonnes** : Projet · Jalon · Chef de projet · Direction · Échéance initiale · Échéance de référence · Dérive (j) · % Avancement · Statut · Cause de retard · **Météo globale projet** (icône, pour le croisement). Colonnes **triables** (cf. tri général, §5/§6).
- **Sélecteur d'axe de vue** (un seul actif à la fois) :
  - **En retard** : jalons en retard.
  - **À risque imminent** : jalon actif, (échéance de référence ≤ date d'obs + 30 j **ou** déjà en retard) **et** (% ≤ 25 **ou** statut ∈ {Non commencé, En retard}).
  - **Échéance ≤ 30 j / ≤ 60 j / ≤ 90 j** : jalons **à venir** dont l'échéance de référence ≤ date d'obs + N jours (**bandes cumulatives**).
  - **Replanifiés** : jalons replanifiés.
  - **Tous** : tous les jalons retenus.

### 11.5 Cartographie du risque
- **Répartition par statut** : effectif par valeur de statut (y compris vide).
- **Concentration** : nombre de jalons **en retard** et **à risque imminent**, agrégés **par Direction** et **par chef de projet**.
- **Risques masqués** : projets dont la **météo globale est verte** et qui portent **≥ 1 jalon en retard** (risque sous-déclaré au niveau projet).

### 11.6 Cohérence des données jalons
Les contrôles qualité propres aux jalons sont restitués dans l'écran **Risk Management** (§7.6.2), et non sur cet écran, conformément au scénario retenu (mutualisation de la zone « cohérence des données »).

---

## 12. Écran « Paramétrage »

**Accès :** une **icône engrenage (⚙)** dans la barre d'application ouvre le panneau **« Paramétrage »**, **hors navigation principale** (il ne figure pas parmi les onglets d'écrans de données). Ce panneau a vocation à regrouper l'ensemble des **valeurs de paramétrage** de l'outil ; sa première section concerne les filtres d'entrée.

### 12.1 Section « Filtres d'entrée »

- **Intitulé projet** : saisie d'une **liste de valeurs** sous forme d'étiquettes (ajout / suppression). Correspondance `%valeur%` insensible à la casse, logique OU (cf. §4). Pré-remplie avec « AEO » et « PIAT ». Liste vide → critère non appliqué.
- **Suivi dans Pulse** : sélecteur **« Oui » / « ALL »** (cf. §4). Valeur par défaut : « Oui ».

### 12.2 Comportement

- **Application dynamique** : toute modification est prise en compte **immédiatement sur tous les écrans**, sans réimport (le filtrage s'opère à l'affichage, cf. §3.1 / §4).
- **Persistance** : les paramètres font partie de l'**état de l'outil** — mémorisés en continuité automatique (IndexedDB) et inclus dans les fichiers **Sauvegarder / Historiser** (`DB.json`), donc restaurés à la réouverture.

---

## 13. Reste à spécifier

- Autres écrans / vues envisagés : fiche projet détaillée, vue par direction, etc. *(à définir)*.
- Destinataires et modalités de restitution / diffusion.

---

*Document de travail — à valider avant lancement du développement (« Go »).*
