# 📐 Système d'Analyse des Lignes de Bus - Équations et Calculs

## Vue d'Ensemble

Ce document détaille toutes les équations et méthodologies de calcul utilisées dans le système d'analyse des lignes de bus de Marrakech. Le système traite 67 lignes de bus sur la période 2026-2035 avec trois saisons (Hiver, Été, Ramadan) et trois types de jours (Lundi-Vendredi, Samedi, Dimanche-Fêtes).

---

## 📊 Paramètres Configurables du Réseau

| Paramètre | Description | Valeur par Défaut | Unité |
|-----------|-------------|-------------------|-------|
| **Tt+Td** | Temps de retournement total | 10 | minutes |
| **v_hlp** | Vitesse hors ligne passagers (HLP) | 25 | km/h |
| **v_tech** | Vitesse technique | 20 | km/h |
| **r_FMDS** | Ratio technique maintenance | 0.05 | ratio |
| **r_ACC** | Ratio accidentologie | 0.01 | ratio |
| **α (alpha)** | Multiplicateur heures payées | 1.10 | ratio |
| **β (beta)** | Réserve conducteurs | 0.12 | ratio |
| **γ (gamma)** | Ratio véhicules de réserve | 0.12 | ratio |
| **h_poste** | Durée du poste conducteur | 7.5 | heures |
| **C** | Capacité par bus | 85 | passagers |
| **δ (delta)** | Distance dépôt (moyenne) | variable | km |

---

## 🔢 Section 1 : Temps de Cycle et Flotte

### 1.1 Temps de Cycle (TC)

Le temps de cycle représente le temps total nécessaire pour qu'un bus effectue un aller-retour complet sur la ligne.

```
TC = temps_aller + temps_retour + Tt+Td
```

**Où :**
- `temps_aller` = Temps de parcours dans le sens aller (minutes)
- `temps_retour` = Temps de parcours dans le sens retour (minutes)
- `Tt+Td` = Temps de retournement aux terminus (10 minutes)

**Exemple :**
- Temps aller : 29 minutes
- Temps retour : 29 minutes
- Tt+Td : 10 minutes
- **TC = 29 + 29 + 10 = 68 minutes**

### 1.2 Buses Nécessaires en Période de Pointe

Le nombre de bus requis pendant les périodes de pointe dépend du temps de cycle et de la fréquence de service.

```
Bus_pointe = ⌈TC / f_pointe⌉
```

**Où :**
- `TC` = Temps de cycle (minutes)
- `f_pointe` = Fréquence en période de pointe (minutes entre deux bus)
- `⌈...⌉` = Fonction plafond (arrondi au supérieur)

**Exemple :**
- TC : 68 minutes
- Fréquence pointe : 15 minutes
- **Bus_pointe = ⌈68 / 15⌉ = ⌈4.53⌉ = 5 bus**

### 1.3 Buses Nécessaires en Période Creuse

```
Bus_vallee = ⌈TC / f_vallee⌉
```

**Où :**
- `f_vallee` = Fréquence en période creuse (minutes)

### 1.4 Nombre Maximum de Buses en Service

```
Bus_max = max(Bus_pointe, Bus_vallee)
```

Le système prend le maximum entre les besoins en pointe et en vallée.

### 1.5 Parc Affecté (Flotte Totale)

Le parc affecté inclut les véhicules de réserve pour maintenance et accidents.

```
Parc_affecté = ⌈Bus_max × (1 + γ)⌉
```

**Où :**
- `γ` = Ratio de réserve de flotte (0.12 = 12%)

**Exemple :**
- Bus_max : 5 bus
- γ : 0.12
- **Parc_affecté = ⌈5 × 1.12⌉ = ⌈5.6⌉ = 6 bus**

---

## 🚌 Section 2 : Calcul des Expéditions (Voyages)

### 2.1 Durée des Périodes de Service

Pour chaque période (pointe ou vallée) :

```
Durée_période = Fin_période - Début_période
```

### 2.2 Nombre d'Expéditions par Période

```
Expeditions_période = Durée_période / Fréquence_période
```

**Exemple :**
- Période de pointe matin : 07:00 - 09:00 (120 minutes)
- Fréquence pointe : 15 minutes
- **Expeditions_pointe = 120 / 15 = 8 expéditions**

### 2.3 Logique Spéciale pour le Ramadan

Pendant le Ramadan, trois périodes de pointe sont définies :

1. **Matin** : 07:00 - 09:00 (période de pointe standard)
2. **Avant Iftar** : 16:00 - 18:00 (2h avant la rupture du jeûne)
3. **Après Iftar** : 21:00 - 23:00 (2h après la rupture)

La fréquence de pointe s'applique à ces trois périodes. Le reste du temps utilise la fréquence vallée.

### 2.4 Total des Expéditions Quotidiennes

```
Trips_AB = Σ(Expeditions_pointe) + Σ(Expeditions_vallee)
Trips_BA = Trips_AB
Trips_tot = Trips_AB + Trips_BA
```

**Où :**
- `Trips_AB` = Voyages dans le sens A → B
- `Trips_BA` = Voyages dans le sens B → A
- `Trips_tot` = Total des voyages quotidiens

---

## 📏 Section 3 : Calcul des Kilomètres

### 3.1 Kilomètres Commerciaux

Les kilomètres commerciaux représentent la distance parcourue avec des passagers.

```
Km_com = 2 × longueur_ligne × Trips_AB
```

**Où :**
- `longueur_ligne` = Longueur de la ligne en km
- Facteur 2 pour l'aller-retour

**Exemple :**
- Longueur ligne : 8.7 km
- Trips_AB : 51 voyages
- **Km_com = 2 × 8.7 × 51 = 887.4 km**

### 3.2 Distance au Dépôt (δ)

```
δ = (Distance_origine_dépôt + Distance_destination_dépôt) / 2
```

**Où :**
- Distance_origine_dépôt = Moyenne entre max et min
- Distance_destination_dépôt = Moyenne entre max et min

### 3.3 Kilomètres Hors Ligne Passagers (HLP)

Les kilomètres HLP représentent les trajets à vide entre le dépôt et les terminus.

```
Km_HLP = δ × (2 × Bus_max)
```

**Explication :** Chaque bus fait un aller-retour au dépôt (facteur 2).

**Exemple :**
- δ : 10 km
- Bus_max : 5 bus
- **Km_HLP = 10 × (2 × 5) = 100 km**

### 3.4 Kilomètres Techniques - Maintenance (FMDS)

```
Km_tech_FMDS = Km_com × r_FMDS
```

**Où :**
- `r_FMDS` = Ratio maintenance (0.05 = 5%)

### 3.5 Kilomètres Techniques - Accidentologie

```
Km_accid = Km_com × r_ACC
```

**Où :**
- `r_ACC` = Ratio accidents (0.01 = 1%)

### 3.6 Kilomètres Techniques Totaux

```
Km_tech = Km_tech_FMDS + Km_accid
Km_tech = Km_com × (r_FMDS + r_ACC)
Km_tech = Km_com × (0.05 + 0.01)
Km_tech = Km_com × 0.06
```

### 3.7 Kilomètres Totaux

```
Km_tot = Km_com + Km_HLP + Km_tech
```

**Exemple complet :**
- Km_com : 887.4 km
- Km_HLP : 100 km
- Km_tech : 53.2 km (887.4 × 0.06)
- **Km_tot = 887.4 + 100 + 53.2 = 1,040.6 km**

---

## ⏱️ Section 4 : Calcul des Heures

### 4.1 Heures de Charge

Les heures de charge représentent le temps de conduite avec passagers.

```
H_charge = (temps_aller × Trips_AB + temps_retour × Trips_BA) / 60
```

**Conversion en heures (division par 60).**

**Exemple :**
- Temps aller : 29 minutes
- Temps retour : 29 minutes
- Trips_AB = Trips_BA : 51 voyages
- **H_charge = (29 × 51 + 29 × 51) / 60 = 2,958 / 60 = 49.3 heures**

### 4.2 Heures Hors Ligne Passagers (HLP)

```
H_HLP = Km_HLP / v_hlp
```

**Où :**
- `v_hlp` = Vitesse HLP (25 km/h)

**Exemple :**
- Km_HLP : 100 km
- v_hlp : 25 km/h
- **H_HLP = 100 / 25 = 4 heures**

### 4.3 Heures Techniques

```
H_tech = Km_tech / v_tech
```

**Où :**
- `v_tech` = Vitesse technique (20 km/h)

**Exemple :**
- Km_tech : 53.2 km
- v_tech : 20 km/h
- **H_tech = 53.2 / 20 = 2.66 heures**

### 4.4 Heures Totales

```
H_total = H_charge + H_HLP + H_tech
```

**Exemple :**
- **H_total = 49.3 + 4 + 2.66 = 55.96 heures**

### 4.5 Heures Payées

Les heures payées incluent un coefficient pour les pauses, préparations, etc.

```
H_payees = H_total × α
```

**Où :**
- `α` = Multiplicateur (1.10 = 10% supplémentaire)

**Exemple :**
- H_total : 55.96 heures
- α : 1.10
- **H_payees = 55.96 × 1.10 = 61.56 heures**

---

## 👨‍✈️ Section 5 : Calcul des Conducteurs (ETP)

### 5.1 Équivalent Temps Plein (ETP)

```
ETP = H_payees / h_poste
```

**Où :**
- `h_poste` = Durée du poste (7.5 heures)

**Exemple :**
- H_payees : 61.56 heures
- h_poste : 7.5 heures
- **ETP = 61.56 / 7.5 = 8.21 conducteurs**

### 5.2 ETP avec Réserve

La réserve compense les congés, absences, formations, etc.

```
ETP_res = ETP × (1 + β)
```

**Où :**
- `β` = Ratio de réserve (0.12 = 12%)

**Exemple :**
- ETP : 8.21
- β : 0.12
- **ETP_res = 8.21 × 1.12 = 9.20 conducteurs**

---

## 🚏 Section 6 : Capacité en Heure de Pointe

### 6.1 Offre en Pointe (Voyages par Heure)

```
O_pointe = 60 / f_pointe
```

**Exemple :**
- Fréquence pointe : 15 minutes
- **O_pointe = 60 / 15 = 4 bus par heure**

### 6.2 Passagers par Heure en Pointe par Direction (PPHPD)

```
PPHPD = C × O_pointe
```

**Où :**
- `C` = Capacité du bus (85 passagers)

**Exemple :**
- C : 85 passagers
- O_pointe : 4 bus/heure
- **PPHPD = 85 × 4 = 340 passagers/heure**

---

## 📅 Section 7 : Agrégations Annuelles

### 7.1 Méthodologie

Le système calcule les métriques annuelles sur la période 2026-2035 (10 ans) en utilisant les données du calendrier qui spécifient le nombre de jours par saison et type de jour.

### 7.2 Structure du Calendrier

Pour chaque année :
- **Hiver (H)** : Jours LaV (Lundi-Vendredi), S (Samedi), DF (Dimanche-Fêtes)
- **Été (Ete)** : Jours LaV, S, DF
- **Ramadan** : Jours LaV, S, DF

### 7.3 Calcul des Totaux sur la Période

Pour chaque combinaison (année, saison, type de jour) :

```
Total_période = Σ(Métrique_quotidienne × Nombre_jours)
```

**Exemple pour les voyages :**
```
Trips_total_2026-2035 = Σ(Trips_quotidien × N_jours)
```

Pour la ligne A01 en Hiver LaV 2026 :
- Trips quotidien : 102 voyages
- Nombre de jours : 187
- Contribution : 102 × 187 = 19,074 voyages

### 7.4 Calcul des Moyennes Annuelles

```
Moyenne_annuelle = Total_période / Nombre_années
```

**Où :**
- `Nombre_années` = 10 (2026-2035)

**Exemple :**
- Total voyages sur 10 ans : 328,540 voyages
- **Moyenne annuelle = 328,540 / 10 = 32,854 voyages/an**

### 7.5 Calcul de l'ETP Annuel

```
ETP_annuel = Σ(H_payees × N_jours) / h_poste
```

Cette formule agrège toutes les heures payées sur l'année puis les divise par la durée du poste.

### 7.6 Flotte Maximale Annuelle

```
Bus_max_annuel = max(Bus_max_quotidien) sur toutes les périodes
Parc_affecté_annuel = ⌈Bus_max_annuel × (1 + γ)⌉
```

La flotte est dimensionnée pour satisfaire le jour le plus chargé de l'année.

---

## 📊 Section 8 : Métriques Réseau

### 8.1 Agrégation au Niveau du Réseau

Les métriques réseau agrègent les valeurs de toutes les lignes :

```
Réseau_total = Σ(Ligne_i) pour i = 1 à 67
```

### 8.2 Métriques Clés du Réseau

1. **Flotte Totale** : Somme des parcs affectés de toutes les lignes
2. **Voyages Annuels** : Somme des voyages moyens annuels
3. **Kilomètres Annuels** : Somme des km moyens annuels
4. **Heures Annuelles** : Somme des heures moyennes annuelles
5. **Conducteurs Total** : Somme des ETP de toutes les lignes

---

## 🔍 Section 9 : Traitement des Données Manquantes

Le système applique des valeurs par défaut pour garantir la robustesse :

### 9.1 Heures de Service

```
Si debut_service = None → debut_service = 0 minutes (00:00)
Si fin_service = None → fin_service = 1439 minutes (23:59)
```

### 9.2 Fréquences

```
Si f_pointe = None ou 0 → f_pointe = 15 minutes
Si f_vallee = None ou 0 → f_vallee = 15 minutes
```

### 9.3 Périodes de Pointe

```
Si periode_pointe = None → Pas de période de pointe définie
```

---

## 📈 Exemples de Calcul Complet

### Exemple : Ligne A01 - Aéroport – Jamâa El Fna (Hiver, Lundi-Vendredi)

#### Données d'Entrée
- Longueur : 8.7 km
- Temps aller : 29 min
- Temps retour : 29 min
- Service : 06:00 - 23:00 (17h = 1020 min)
- Fréquence : 20 min (pointe et vallée identiques)
- Distance dépôt moyenne : 10 km

#### Calculs

**1. Temps de Cycle**
```
TC = 29 + 29 + 10 = 68 minutes
```

**2. Buses Nécessaires**
```
Bus_max = ⌈68 / 20⌉ = ⌈3.4⌉ = 4 bus
Parc_affecté = ⌈4 × 1.12⌉ = ⌈4.48⌉ = 5 bus
```

**3. Expéditions**
```
Durée service = 1020 minutes
Trips_AB = 1020 / 20 = 51 voyages
Trips_tot = 51 × 2 = 102 voyages
```

**4. Kilomètres**
```
Km_com = 2 × 8.7 × 51 = 887.4 km
Km_HLP = 10 × (2 × 4) = 80 km
Km_tech = 887.4 × 0.06 = 53.2 km
Km_tot = 887.4 + 80 + 53.2 = 1,020.6 km
```

**5. Heures**
```
H_charge = (29 × 51 + 29 × 51) / 60 = 49.3 h
H_HLP = 80 / 25 = 3.2 h
H_tech = 53.2 / 20 = 2.66 h
H_total = 49.3 + 3.2 + 2.66 = 55.16 h
H_payees = 55.16 × 1.10 = 60.68 h
```

**6. Conducteurs**
```
ETP = 60.68 / 7.5 = 8.09 conducteurs
ETP_res = 8.09 × 1.12 = 9.06 conducteurs
```

**7. Capacité**
```
O_pointe = 60 / 20 = 3 bus/heure
PPHPD = 85 × 3 = 255 passagers/heure
```

---

## 📝 Notes Techniques

### Précision des Calculs
- Tous les calculs intermédiaires utilisent des nombres décimaux
- Les arrondis sont appliqués uniquement pour l'affichage final
- La fonction plafond (⌈⌉) est utilisée pour les quantités discrètes (bus, conducteurs)

### Validation
- Les métriques quotidiennes sont calculées pour chaque combinaison saison/type de jour
- Les agrégations annuelles utilisent les données de calendrier réelles (2026-2035)
- Les totaux de 10 ans sont divisés par 10 pour obtenir les moyennes annuelles

### Cohérence
- Les formules suivent strictement le document de spécification
- Les valeurs par défaut assurent la robustesse face aux données incomplètes
- Tous les calculs sont reproductibles et traçables

---

## 🎯 Résumé des Formules Principales

| Métrique | Formule |
|----------|---------|
| **Temps de cycle** | `TC = t_aller + t_retour + 10` |
| **Buses pointe** | `⌈TC / f_pointe⌉` |
| **Parc affecté** | `⌈Bus_max × 1.12⌉` |
| **Voyages totaux** | `2 × (Durée / Fréquence)` |
| **Km commerciaux** | `2 × longueur × Trips_AB` |
| **Km HLP** | `δ × (2 × Bus_max)` |
| **Km techniques** | `Km_com × 0.06` |
| **Km totaux** | `Km_com + Km_HLP + Km_tech` |
| **Heures charge** | `(t_aller × Trips_AB + t_retour × Trips_BA) / 60` |
| **Heures payées** | `H_total × 1.10` |
| **ETP** | `H_payees / 7.5` |
| **ETP avec réserve** | `ETP × 1.12` |
| **PPHPD** | `85 × (60 / f_pointe)` |

---

**Document généré par le Système d'Analyse des Lignes de Bus**  
**Version 1.0 - 2025-11-09**
