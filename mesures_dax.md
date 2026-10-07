# Mesures DAX · DistriTech

Toutes les mesures sont rangées dans la table `_Mesures`.

## 1. Chiffre d'affaires et objectifs

```DAX
CA_Total = SUM(Fact_Ventes[Montant_HT])

Objectif_CA_Total = SUM(Fact_Objectifs[Objectif_CA])

-- CA limité aux couples magasin × mois qui ont un objectif,
-- pour comparer le réel et l'objectif sur le même périmètre
-- (Mois_Num = numéro du mois 1 à 12 dans Dim_Date)
CA_Sous_Objectif =
SUMX(
    SUMMARIZE(Fact_Objectifs, Dim_Magasins[ID_Magasin], Dim_Date[Annee], Dim_Date[Mois_Num]),
    [CA_Total]
)

Ecart_Objectif = [CA_Sous_Objectif] - [Objectif_CA_Total]

Pct_Realisation = DIVIDE([CA_Sous_Objectif], [Objectif_CA_Total])

-- Barres du graphique combiné : CA total quand il n'y a pas d'objectif (2014)
CA_Reel_Graph = COALESCE([CA_Sous_Objectif], [CA_Total])
```

## 2. Time intelligence

```DAX
CA_N_1 = CALCULATE([CA_Total], SAMEPERIODLASTYEAR(Dim_Date[Date]))

Croissance_N1 = DIVIDE([CA_Total] - [CA_N_1], [CA_N_1])

CA_YTD = TOTALYTD([CA_Total], Dim_Date[Date])

CA_YTD_N1 = CALCULATE([CA_YTD], SAMEPERIODLASTYEAR(Dim_Date[Date]))

Croissance_YTD = DIVIDE([CA_YTD] - [CA_YTD_N1], [CA_YTD_N1])
```

## 3. Mise en forme conditionnelle

```DAX
Couleur_Objectif =
VAR p = [Pct_Realisation]
RETURN SWITCH(TRUE(), ISBLANK(p), BLANK(), p >= 1, "#2E9E5B", "#D64545")

Couleur_Croissance =
VAR c = [Croissance_N1]
RETURN SWITCH(TRUE(), ISBLANK(c), BLANK(), c >= 0, "#2E9E5B", "#D64545")
```

## 4. Affichage (cartes KPI)

```DAX
CA_Affiche = FORMAT([CA_Total] / 1000, "#,0") & " k$"

Pct_Affiche =
IF(ISBLANK([Pct_Realisation]), "n.d.", FORMAT([Pct_Realisation], "0.0 %"))

Croissance_Affiche =
IF(ISBLANK([Croissance_N1]), "n.d.", FORMAT([Croissance_N1], "+0.0 %;-0.0 %"))
```

## 5. Titres et sous-titres dynamiques

```DAX
Sous_Titre_CA =
VAR a = SELECTEDVALUE(Dim_Date[Annee])
RETURN IF(ISBLANK(a), "Toutes années · hors taxes", "Exercice " & a & " · hors taxes")

Sous_Titre_Objectif =
VAR e = [Ecart_Objectif]
RETURN IF(ISBLANK(e), "Pas d'objectif sur la période",
    "Écart : " & IF(e >= 0, "+", "") & FORMAT(e / 1000, "#,0") & " k$ vs objectif")

Sous_Titre_N1 =
VAR a = SELECTEDVALUE(Dim_Date[Annee])
RETURN IF(ISBLANK([CA_N_1]), "Pas de données N-1",
    "CA " & IF(ISBLANK(a), "N-1", a - 1) & " : " & FORMAT([CA_N_1] / 1000, "#,0") & " k$")

Sous_Titre_Graph = SELECTEDVALUE(Dim_Date[Annee], "2014-2017") & " · en milliers de $"

Sous_Titre_Region =
VAR a = SELECTEDVALUE(Dim_Date[Annee], "2014-2017")
RETURN IF(ISBLANK([Objectif_CA_Total]),
    a & " · pas d'objectif (année de référence)",
    a & " · objectif = 100 %")

Info_Donnees =
"Données au " & FORMAT(MAX(Fact_Ventes[Date_Commande]), "dd/mm/yyyy")
    & " · Croissance cumulée (YTD) : " & FORMAT([Croissance_YTD], "+0.0 %;-0.0 %")

Contexte_Filtres =
"Exercice " & SELECTEDVALUE(Dim_Date[Annee], "2014-2017")

Titre_Infobulle =
"Top 3 produits · " & SELECTEDVALUE(Dim_Magasins[Etat], "Tous magasins")
```
