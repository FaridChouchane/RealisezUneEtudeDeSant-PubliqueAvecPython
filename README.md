# 🌍 Étude de santé publique — Sous-nutrition mondiale

<div align="center">

<img src="assets/fao.png" alt="FAO" width="600"/>

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-1.x-150458?style=flat-square&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-1.x-013243?style=flat-square&logo=numpy&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat-square&logo=jupyter&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)
![Data](https://img.shields.io/badge/Data-FAO%20%2F%20FAOSTAT-blue?style=flat-square)

**Analyse de la sous-nutrition mondiale à partir des données open data de la FAO**

*Projet réalisé dans le cadre de la formation Data Analyst — OpenClassrooms*

</div>

---

## 📁 Structure du projet

```
📦 FAO_Chouchane_Farid/
├── 📄 README.md
├── 📂 assets/
│   └── fao.png
├── 📓 Chouchane_Farid_1_notebook_032022.ipynb
└── 📂 data/
    ├── aide_alimentaire.csv
    ├── dispo_alimentaire.csv
    ├── population.csv
    └── sous_nutrition.csv
```

---

## 📋 Sommaire

- [Contexte](#-contexte)
- [Sources de données](#-sources-de-données)
- [Installation](#-installation)
- [Analyses réalisées](#-analyses-réalisées)
- [Résultats clés](#-résultats-clés)
- [Stack technique](#️-stack-technique)
- [Compétences démontrées](#-compétences-démontrées)

---

## 🎯 Contexte

En tant que data analyst au sein d'une équipe de chercheurs de la **FAO (Food and Agriculture Organization of the United Nations)**, l'objectif est de produire un panorama de l'état de la **sous-nutrition dans le monde** à partir des données FAOSTAT pour l'année **2017**.

L'étude répond à deux séries de demandes :
- **Marc** (responsable d'équipe) : indicateurs globaux sur la disponibilité alimentaire mondiale
- **Mélanie** (chercheuse) : analyse fine par pays pour identifier les zones les plus en difficulté

> 📅 Données : **2017** pour la disponibilité alimentaire · **2013–2017** pour l'aide alimentaire

---

## 📦 Sources de données

| Fichier | Contenu | Unité |
|---------|---------|-------|
| `dispo_alimentaire.csv` | Disponibilité alimentaire par pays et produit | kcal/personne/jour · milliers de tonnes |
| `sous_nutrition.csv` | Personnes en sous-alimentation par pays | millions d'habitants |
| `population.csv` | Population par pays et par année | milliers d'habitants |
| `aide_alimentaire.csv` | Aide alimentaire reçue par pays depuis 2013 | tonnes |

> Données issues de [FAOSTAT](https://www.fao.org/faostat/fr/) — librement téléchargeables

---

## ⚙️ Installation

### Prérequis

```bash
pip install pandas numpy jupyter
```

### Lancer le notebook

```bash
# Placer les fichiers CSV dans le même dossier que le notebook
jupyter notebook Chouchane_Farid_1_notebook_032022.ipynb
```

### Chargement des données

```python
import pandas as pd
import numpy as np

aide_alim  = pd.read_csv('aide_alimentaire.csv')
dispo_alim = pd.read_csv('dispo_alimentaire.csv')
pop        = pd.read_csv('population.csv')
sous_nut   = pd.read_csv('sous_nutrition.csv')
```

> ⚠️ Les fichiers CSV doivent être placés dans le même dossier que le notebook (chemins relatifs).

---

## 📊 Analyses réalisées

### Partie 1 — Demandes de Marc (indicateurs mondiaux 2017)

#### 1.1 — Proportion de personnes en sous-nutrition

```python
# Les années sont fournies sous forme d'intervalles (ex: '2012-2014').
# On les remplace par l'année médiane correspondante.
sous_nut.replace(
    ['2012-2014', '2013-2015', '2014-2016', '2015-2017', '2016-2018', '2017-2019'],
    ['2013',      '2014',      '2015',      '2016',      '2017',      '2018'],
    inplace=True
)

sous_nut.fillna(0, inplace=True)
sous_nut['Valeur'].replace({'<0.1': 0.1}, inplace=True)
sous_nut['Valeur'] = sous_nut['Valeur'].astype('float64')
sous_nut['Année']  = sous_nut['Année'].astype(int)

# Population mondiale 2017 (données en milliers → ×1 000)
pop_mondiale_2017 = pop.loc[pop['Année'] == 2017, 'Valeur'].sum() * 1_000

# Sous-nutrition 2017 (données en millions → ×1 000 000)
sous_nut_2017 = sous_nut.loc[sous_nut['Année'] == 2017, 'Valeur'].sum() * 1_000_000

prop_ssnut_2017 = round(sous_nut_2017 / pop_mondiale_2017 * 100, 2)
```

> **Résultat : 7,12 % de la population mondiale — soit 537 700 000 personnes — étaient en sous-nutrition en 2017**

---

#### 1.2 — Capacité nourricière mondiale (toutes origines)

```python
# Jointure : disponibilité alimentaire × population par pays
pop_2017   = pop.loc[pop['Année'] == 2017]
dispo_alim = pd.merge(dispo_alim, pop_2017, how='left', on='Zone')

# kcal disponibles par jour pour la population de chaque pays
dispo_alim['kcal/jour/population'] = (
    dispo_alim['Valeur']
    * dispo_alim['Disponibilité alimentaire (Kcal/personne/jour)']
)

# Nombre de personnes théoriquement nourries (base : 2 400 kcal/jour/adulte)
capacite_mondiale = round(dispo_alim['kcal/jour/population'].sum() / 2400 * 1_000)
```

> **Résultat : ~8,7 milliards de personnes pouvaient être nourries, soit 1,15× la population mondiale**

---

#### 1.3 — Capacité nourricière végétale uniquement

```python
# Filtrage sur les produits d'origine végétale uniquement
dispo_alim_vege = dispo_alim.loc[dispo_alim['Origine'] == 'vegetale'].copy()

dispo_alim_vege['kcal_vege/jour/population'] = (
    dispo_alim_vege['Valeur_x']
    * dispo_alim_vege['Disponibilité alimentaire (Kcal/personne/jour)']
)

capacite_vege = round(dispo_alim_vege['kcal_vege/jour/population'].sum() / 2400 * 1_000)
```

> **Résultat : ~7,2 milliards avec la seule dispo végétale, soit 0,95× la population mondiale**

---

#### 1.4 — Répartition de la disponibilité intérieure

```python
dis_int_tot  = dispo_alim['Disponibilité intérieure'].sum()

# Part alimentation animale
alim_animale = dispo_alim['Aliments pour animaux'].sum()
prop_animale = round(alim_animale / dis_int_tot * 100)

# Part pertes
pertes       = dispo_alim['Pertes'].sum()
prop_pertes  = round(pertes / dis_int_tot * 100)

# Part alimentation humaine (méthode suggérée par Julien : dispo − tout le reste)
part_humain  = (
    dis_int_tot
    - pertes
    - alim_animale
    - dispo_alim['Semences'].sum()
    - dispo_alim['Traitement'].sum()
    - dispo_alim['Autres Utilisations'].sum()
)
prop_humain  = round(part_humain / dis_int_tot * 100)
```

| Utilisation | Proportion |
|-------------|-----------|
| 🐄 Alimentation animale | **13 %** |
| ♻️ Pertes | **5 %** |
| 🍽️ Alimentation humaine | **49 %** |

---

### Partie 2 — Demandes de Mélanie (analyse par pays)

#### 2.1 — Pays avec la proportion de sous-alimentés la plus forte (2017)

```python
# Jointure population × sous-nutrition par pays
merge1 = pd.merge(
    pop.loc[pop['Année'] == 2017],
    sous_nut.loc[sous_nut['Année'] == 2017],
    how='outer',
    on=['Zone', 'Année'],
    suffixes=('_pop', '_ssnut')
)

# Conversion des unités
merge1['Valeur_ssnut'] = merge1['Valeur_ssnut'] * 1_000_000
merge1['Valeur_pop']   = merge1['Valeur_pop']   * 1_000

# Proportion de sous-alimentés par pays
merge1['prop_sous_nutrition'] = merge1['Valeur_ssnut'] / merge1['Valeur_pop']

# Filtre : pays > 1 million d'habitants (résultats plus représentatifs)
top10_ssnut = (
    merge1.loc[merge1['Valeur_pop'] > 1_000_000]
    .sort_values('prop_sous_nutrition', ascending=False)
    .head(10)[['Zone', 'prop_sous_nutrition']]
    .reset_index(drop=True)
)
```

| Pays | Proportion sous-alimentés |
|------|--------------------------|
| Haïti | **48,3 %** |
| Corée du Nord | 47,2 % |
| Madagascar | 41,1 % |
| … | … |

---

#### 2.2 — Pays ayant le plus bénéficié d'aide alimentaire depuis 2013

```python
# Somme de l'aide reçue par pays bénéficiaire (toutes années confondues)
top10_aide = (
    aide_alim
    .groupby('Pays bénéficiaire')['Valeur']
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .reset_index()
    .rename(columns={'Valeur': 'Aide totale reçue (tonnes)'})
)
```

| Pays bénéficiaire | Aide reçue (tonnes) |
|-------------------|---------------------|
| République arabe syrienne | **1 858 943** |
| Éthiopie | 1 381 294 |
| Yémen | 1 206 484 |
| Soudan du Sud | 695 248 |

---

#### 2.3 — Pays avec la plus faible disponibilité alimentaire par habitant

```python
# Disponibilité moyenne par pays (kcal/personne/jour)
top10_faible_dispo = (
    dispo_alim
    .groupby('Zone')['Disponibilité alimentaire (Kcal/personne/jour)']
    .mean()
    .reset_index()
    .sort_values('Disponibilité alimentaire (Kcal/personne/jour)', ascending=True)
    .head(10)
    .reset_index(drop=True)
)
```

| Pays | Kcal/personne/jour |
|------|-------------------|
| République centrafricaine | **1 879** |
| Zambie | … |

> ⚠️ Rappel : le seuil recommandé par la FAO est de **2 400 kcal/jour/personne**

---

## 📈 Résultats clés

| Indicateur | Valeur |
|------------|--------|
| 🌍 Population mondiale 2017 | **7,55 milliards** |
| 🍽️ Personnes en sous-nutrition | **537,7 millions (7,12 %)** |
| 📊 Capacité nourricière mondiale | **8,7 milliards (×1,15 la pop.)** |
| 🌱 Capacité nourricière végétale | **7,2 milliards (×0,95 la pop.)** |
| 🐄 Part alimentation animale | **13 % de la dispo. intérieure** |
| ♻️ Part pertes alimentaires | **5 % de la dispo. intérieure** |
| 🍞 Part alimentation humaine | **49 % de la dispo. intérieure** |
| 🚨 Pays le plus touché | **Haïti — 48,3 % de sous-alimentés** |
| 🆘 1er bénéficiaire d'aide | **Syrie — 1 858 943 tonnes** |
| 📉 Pays avec le moins de dispo. | **Rép. centrafricaine — 1 879 kcal/j** |

---

## 🛠️ Stack technique

| Outil | Usage |
|-------|-------|
| **Python 3** | Langage principal |
| **Pandas** | Manipulation des DataFrames, jointures, agrégations |
| **NumPy** | Calculs numériques |
| **Jupyter Notebook** | Environnement de développement et présentation |

---

## ✅ Compétences démontrées

- [x] Création d'un environnement de développement Python
- [x] Import et nettoyage de données multi-sources (CSV)
- [x] Manipulation avancée de DataFrames (`merge`, `groupby`, `fillna`, `astype`)
- [x] Conversion et normalisation d'unités hétérogènes
- [x] Calculs statistiques et indicateurs agrégés
- [x] Filtrage, tri et sélection de données (`.loc`, `.head`, `.sort_values`)
- [x] Présentation des résultats sous forme de notebook structuré et documenté
- [x] Communication d'analyses à des parties prenantes non-techniques

---

## 👤 Auteur

**Farid Chouchane** — Data Analyst  
Formation Data Analyst · OpenClassrooms

---

<div align="center">
<sub>Données open data — FAOSTAT · FAO · Nations Unies · Périmètre : 2017 · Monde</sub>
</div>
