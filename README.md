# TP3 - NoSQL, transformations et Data Lakehouse

## Présentation du TP

Dans ce TP, j'ai travaillé sur des données de ventes et de produits d'un magasin.

Le but était de voir comment on peut utiliser PySpark pour lire des données, vérifier leur qualité, les nettoyer et ensuite les organiser dans un Data Lakehouse.

J'ai travaillé sur Jupyter Notebook, avec Python et PySpark. J'ai aussi utilisé Pandas et PyArrow pour enregistrer les résultats au format Parquet.

Je me suis concentrée sur la partie obligatoire UC-06.

## 1. Les données utilisées

J'ai utilisé deux fichiers du projet Retail Intelligence Platform :

- **store_sales.dat** : contient les ventes réalisées en magasin, avec 667 579 lignes.
- **item.dat** : contient les informations sur les articles, avec 17 820 lignes.

Les fichiers étaient au format `.dat`, sans noms de colonnes, et les valeurs étaient séparées par le caractère `|`.

J'ai commencé par regarder la structure des fichiers, le nombre de lignes et les types de données avant de les charger avec PySpark.

## 2. Exploration et qualité des données

Une fois les fichiers chargés dans Spark, j'ai affiché les premières lignes et les schémas pour mieux comprendre les données.

J'ai ensuite vérifié :

- Les valeurs manquantes.
- Les doublons.
- Les quantités vendues.
- Les prix des articles.
- La correspondance entre les identifiants des ventes et des produits.

J'ai obtenu les résultats suivants :

| Vérification | Résultat |
|---|---|
| Valeurs manquantes dans les ventes | 1 139 |
| Valeurs manquantes dans les articles | 21 203 |
| Doublons complets | 0 |
| Articles sans correspondance | 0 |
| Quantités vendues nulles ou négatives | 0 |

J'ai remarqué une différence entre Pandas et PySpark pour les valeurs manquantes du fichier des articles.

En vérifiant, j'ai trouvé que **3 365 valeurs étaient écrites `N/A`**. Pandas les considérait comme manquantes, alors que Spark les gardait comme du texte.

Je les ai donc remplacées par des valeurs `NULL` pour avoir un traitement plus cohérent.

## 3. Nettoyage et transformation

Après l'exploration, j'ai commencé à préparer les données.

J'ai d'abord renommé les colonnes, car elles s'appelaient simplement `_c0`, `_c1`, etc.

J'ai utilisé les noms du modèle TPC-DS, comme `ss_quantity` pour les quantités et `i_category` pour les catégories des produits.

J'ai également converti les types :

- Les identifiants et les quantités en entiers.
- Les prix et les montants en décimaux.
- Les dates au format date.

J'ai ensuite vérifié que les conversions n'avaient pas créé de nouvelles valeurs nulles.

Pour les ventes, j'ai gardé uniquement celles qui avaient un identifiant d'article et une quantité supérieure à zéro. Au final, aucune ligne n'a été supprimée, puisque toutes respectaient ces conditions.

## 4. Jointure entre les ventes et les articles

J'ai ensuite réalisé une jointure entre les deux fichiers avec les colonnes `ss_item_sk` et `i_item_sk`.

Cela m'a permis d'ajouter des informations sur les produits aux ventes, notamment leur nom, leur catégorie et leur prix.

Avant la jointure, j'avais **667 579 lignes**, et j'ai retrouvé le même nombre après.

J'ai donc pu continuer les analyses sans perdre de ventes.

## 5. Organisation du Data Lakehouse

Pour organiser les résultats, j'ai utilisé le principe des zones Bronze, Silver et Gold.

**Bronze :** correspond aux fichiers de départ, que j'ai conservés sans les modifier.

**Silver :** contient les données nettoyées et les types corrigés. J'ai enregistré deux fichiers Parquet :

- `silver/sales/sales.parquet`
- `silver/products/products.parquet`

**Gold :** contient les résultats de mon analyse. J'ai regroupé les ventes par catégorie de produit pour calculer le nombre de lignes de vente, les quantités vendues et le montant total.

Le résultat a été enregistré dans :

`gold/sales_by_category/sales_by_category.parquet`

## 6. Résultats obtenus

J'ai obtenu neuf catégories de produits.

Voici quelques résultats :

| Catégorie | Montant total |
|---|---:|
| Music | 519 666 693,84 |
| Books | 307 151 503,25 |
| Movies & TV | 74 735 891,89 |
| Electronics | 55 715 087,49 |
| Home & Kitchen | 41 260 511,11 |

On peut voir que les catégories **Music** et **Books** ont les montants de ventes les plus importants dans ce jeu de données.

J'ai aussi enregistré un rapport de qualité dans le fichier `output/rapport_qualite.csv`.

## 7. Comparaison entre CSV et Parquet

J'ai comparé les deux formats en utilisant le fichier des articles.

| | CSV | Parquet |
|---|---:|---:|
| Taille | 6,66 Mo | 3,60 Mo |
| Temps de lecture | 0,2494 s | 0,1112 s |

Pour mes données, le fichier Parquet prend **45,91 % de place en moins** que le CSV.

Il a aussi été plus rapide à lire pendant mon test. Par contre, ce temps peut changer selon l'ordinateur et les conditions d'exécution.

J'ai compris que CSV est pratique pour consulter facilement les données, tandis que Parquet est plus intéressant pour stocker et analyser des données avec leurs types.

## 8. Difficultés rencontrées

La principale difficulté a été la configuration de PySpark sous Windows.

Au début, j'ai eu quelques problèmes avec Java et avec la version de Python utilisée par Spark.

J'ai réussi à initialiser la session Spark en mode local, puis à lire et transformer les fichiers.

J'ai aussi rencontré un problème lors de l'écriture des fichiers Parquet avec Spark, lié à Hadoop sous Windows.

Pour pouvoir avancer, j'ai utilisé PyArrow pour enregistrer les résultats en Parquet après les transformations réalisées avec PySpark.

J'ai ensuite relu les fichiers pour vérifier qu'ils avaient bien été enregistrés.


