# Lumina & Co — Data Marketing & Analytics

Projet de cours DIA2. Lumina & Co est une marque française de cosmétiques vendue
en direct au consommateur, fondée il y a trois ans. Après une forte croissance,
les ventes stagnent depuis 2024. Le CMO vient de recruter une équipe data et
arrive avec trois questions.

> « On a des dizaines de milliers de clients dans notre CRM mais on envoie les
> mêmes emails à tout le monde. C'est un problème ? »

> « On a lancé plusieurs campagnes cette année. On a dépensé un budget
> conséquent. Le social nous dit que tout marche très bien, Google nous dit la
> même chose. Mais je m'attendais à plus de ventes. »

> « Pour les prochaines campagnes, je veux des recommandations basées sur des
> tests, pas sur des intuitions. »

Ce dépôt contient l'analyse qui répond aux deux premières, et prépare la
troisième.

## Les données

**Les fichiers de données ne sont pas versionnés.** Ils pèsent 236 Mo au total,
`transactions.csv` dépasse à lui seul la limite de 100 Mo par fichier imposée par
GitHub, et il s'agit de données clients.

Pour exécuter les notebooks, déposer les quatre fichiers fournis dans
`data/raw/` :

| Fichier | Taille | Contenu | Utilisé par |
|---|---|---|---|
| `customers.csv` | 4,9 Mo | 50 295 clients : pays, dates de premier et dernier achat, métriques agrégées, données déclaratives | notebooks 01 à 04 |
| `transactions.csv` | 142 Mo | 1 613 433 lignes, une par produit facturé | notebooks 01 à 04 |
| `campaigns.csv` | 1 Ko | 6 campagnes marketing et leurs métriques agrégées | notebook 03 |
| `touchpoints.csv` | 100 Mo | 1 080 897 points de contact, reliés par un identifiant de parcours | notebooks 03 et 04 |

Le dossier `data/processed/` reste vide : il est rempli par le notebook 01, qui y
écrit le jeu de données nettoyé consommé par les trois autres.

## Arborescence

```
.
├── data/
│   ├── raw/                      déposer ici les 4 CSV sources
│   └── processed/                rempli par le notebook 01
├── notebooks/
│   ├── 01_eda_nettoyage.ipynb
│   ├── 02_segmentation_rfm.ipynb
│   ├── 03_kpis_canaux.ipynb
│   └── 04_attribution.ipynb
├── requirements.txt
├── .gitignore
└── README.md
```

La séparation entre `raw/` et `processed/` n'est pas cosmétique. La donnée brute
est la seule référence vérifiable : tant qu'elle reste intacte, on peut rejouer
tout le pipeline depuis zéro et montrer d'où vient chaque chiffre. Si on nettoie
un fichier en écrasant l'original, une erreur de traitement devient irrattrapable.

## Installation

Python 3.12 est utilisé plutôt qu'une version plus récente : les paquets
scientifiques y disposent de roues précompilées stables.

macOS et Linux :

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m ipykernel install --user --name lumina-tp1 --display-name "Python (Lumina TP1)"
```

Windows (PowerShell) :

```powershell
py -3.12 -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m ipykernel install --user --name lumina-tp1 --display-name "Python (Lumina TP1)"
```

Ouvrir ensuite les notebooks et sélectionner le kernel « Python (Lumina TP1) ».

## Ordre d'exécution

Le notebook 01 doit être exécuté en premier : il produit les fichiers de
`data/processed/` dont dépendent les trois autres. Les notebooks 02, 03 et 04
sont ensuite indépendants entre eux.

Les notebooks sont versionnés avec leurs sorties et leurs figures. On peut donc
lire l'intégralité des résultats sans disposer des données.

## Ce que contient chaque notebook

### 01 — Exploration et nettoyage

Diagnostic avant tout traitement, puis dix décisions de nettoyage documentées une
par une. Le principe retenu : une suppression est définitive, un indicateur se
discute. Seuls les 445 doublons stricts sont supprimés ; tout le reste est
conservé et qualifié par des drapeaux, ce qui permet de choisir le périmètre au
moment de l'analyse.

Trois découvertes structurelles ont orienté toute la suite. Le fichier de
transactions réunit **deux exports différents** aux clientèles disjointes, un seul
client commun sur 49 402. Les identifiants clients manquants ne correspondent pas
à des achats en invité mais à une perte à l'export, ce qui a permis d'en
récupérer 107 231. Et les métriques agrégées de `customers.csv` ne se réconcilient
pas avec les factures : 169,7 M€ annoncés contre 50,6 M€ calculés.

Se termine sur cinq hypothèses marketing, chacune assortie de ce qui
l'invaliderait.

### 02 — Segmentation RFM

Construction des indicateurs de récence, fréquence et montant, puis scoring de 1
à 5 et découpage en huit segments.

Le point technique de l'étape : le découpage par quintiles **échoue** sur la
fréquence, parce que 29 % des clients ont exactement une commande. Trois
contournements sont testés et comparés dans le notebook ; celui retenu fixe des
paliers métier, moins élégants statistiquement mais défendables devant un
non-technicien.

Les huit segments sont ensuite profilés, priorisés selon un critère explicite, et
dotés chacun d'une recommandation d'action. Une dernière section examine s'il
vaut la peine de les découper plus finement : quatre segments sont sous-segmentés,
quatre ne le sont pas, et les raisons du refus sont écrites.

### 03 — KPIs par canal

Coût par acquisition, coût d'acquisition client, retour sur dépense publicitaire
et taux de conversion, calculés pour les sept canaux.

Le classement change complètement selon la métrique retenue, ce qui montre
qu'aucune ne suffit seule. Deux canaux sortent de toute mesure faute de
conversion directe. Le champ `primary_channel` des rapports de campagne se révèle
désigner le canal le plus diffusé, pas le plus performant. Et le calcul de la
valeur vie client rapportée au coût d'acquisition renverse le classement obtenu
par les métriques classiques.

### 04 — Attribution multicanal

Comparaison des modèles premier contact et dernier contact sur les mêmes
94 366 conversions.

Le même chiffre d'affaires se répartit de deux façons presque opposées selon la
convention de comptage : deux canaux passent de 82 % du crédit à zéro. Le
notebook en tire une recommandation sur le modèle à retenir, et une analyse de ce
qu'on peut conclure quand on ne dispose pas d'une vision croisée des canaux.

## Limites de l'analyse

Documentées dans les notebooks et à reprendre dans toute restitution.

1. L'historique réunit deux exports aux clientèles disjointes. Certains écarts
   entre segments peuvent venir de là plutôt que du comportement des clients.
2. La base est française à 91 %. Les segments décrivent le client français ;
   les appliquer à un autre marché serait une extrapolation.
3. 16 % du chiffre d'affaires n'est rattaché à aucun client. Les analyses par
   client portent sur 84 % de l'activité.
4. La tendance du chiffre d'affaires n'est pas interprétable en l'état : sa
   croissance apparente correspond à la montée en charge du second export.
5. Les données déclaratives sont vides à 62 % au mieux et réparties de façon
   trop uniforme pour être crédibles. Elles ne peuvent pas servir à segmenter.

## Environnement

pandas 3.0.5, numpy 2.5.3, matplotlib 3.11.1, seaborn 0.13.2, pyarrow 25.0.1,
jupyter 1.1.1, ipykernel 7.3.0. Versions épinglées dans `requirements.txt`.
