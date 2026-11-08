# Analyse du Churn Client & Impact Financier  Secteur Télécom

![Dashboard Power BI](captures/visualisation.png)

## Le projet en une phrase

À partir d'un jeu de données Kaggle de 7 043 clients d'un opérateur télécom, ce projet nettoie les données, les structure dans PostgreSQL puis construit un dashboard Power BI qui chiffre en FCFA le manque à gagner lié au départ des clients et identifie les abonnés à relancer en priorité.

## Origine des données

Le dataset utilisé, "Telco Customer Churn", est un jeu de données d'exemple publié par IBM et disponible sur Kaggle. Il décrit 7 043 clients à travers 21 colonnes : type de contrat, service internet, mode de paiement, ancienneté, facturation mensuelle et totale, et une variable cible `Churn` indiquant si le client a quitté l'opérateur.

Ces données sont américaines. Pour illustrer la démarche dans le contexte du marché camerounais (Douala, Yaoundé), les montants ont été convertis en FCFA avec un taux fixe de 1 USD = 600 FCFA. Il s'agit d'une adaptation illustrative, qui ne reflète pas les tarifs réels d'un opérateur local.

## Démarche suivie, étape par étape

### 1. Import et diagnostic des données (Python)

Le fichier CSV est téléchargé depuis Kaggle puis chargé avec Pandas. Avant toute analyse, la structure est vérifiée : dimensions, types de colonnes, valeurs manquantes.

Ce diagnostic révèle un piège : la colonne `TotalCharges` (facturation totale) est lue comme du texte alors qu'elle devrait être numérique. La cause est la présence d'espaces vides (`" "`) saisis à la place des valeurs nulles pour les nouveaux clients, qui ont une ancienneté de 0 mois. Ces valeurs échappent à une détection classique des données manquantes.

### 2. Nettoyage des données (Python)

La colonne `TotalCharges` est convertie en numérique avec `pd.to_numeric(..., errors='coerce')`, ce qui transforme les espaces vides en valeurs manquantes. Celles-ci sont ensuite remplacées par 0, ce qui est cohérent : un client à 0 mois d'ancienneté n'a encore rien payé. Ce choix évite aussi de fausser les moyennes.

Une nouvelle colonne, `MonthlyCharges_FCFA`, est créée en multipliant la facturation mensuelle par 600. Le fichier propre est exporté sous `telecom_churn_propre.csv`.

### 3. Analyse exploratoire (Python)

Des graphiques Seaborn et Matplotlib permettent de valider les premières hypothèses avant de passer à la modélisation. Deux constats ressortent :

- Le taux de churn global atteint 26,5 %, un niveau critique pour un opérateur télécom.
- Les contrats mensuels sans engagement (*Month-to-month*) concentrent la quasi-totalité des départs, tandis que les contrats de 1 ou 2 ans stabilisent le portefeuille.

### 4. Stockage et logique métier (PostgreSQL)

Le fichier propre est chargé dans une base PostgreSQL, qui simule l'environnement de production d'une entreprise. Plutôt que de surcharger Power BI avec des calculs lourds, la logique métier est déportée dans deux vues SQL :

- **`vue_performance_financiere`** : synthèse pour la direction, avec le nombre de clients, le taux de churn, le chiffre d'affaires mensuel actif et le manque à gagner mensuel en FCFA, par type de contrat et de service internet.
- **`vue_alerte_churn_vip`** : liste d'action pour les équipes de relance. Elle isole les clients encore actifs, sous contrat mensuel, dont la facture dépasse la moyenne du portefeuille, triés par montant décroissant.

Une particularité technique : la fonction `ROUND()` de PostgreSQL n'accepte pas les colonnes de type `double precision` avec un nombre de décimales. Les calculs passent donc par un cast explicite `::NUMERIC`.

### 5. Construction du dashboard (Power BI)

Power BI Desktop se connecte au serveur PostgreSQL local en mode Import, ce qui accélère les filtres croisés. Le tableau de bord est organisé en trois sections :

- **Les indicateurs clés** : le manque à gagner mensuel en FCFA et le nombre de clients VIP à sauver.
- **L'explication** : un graphique à barres montrant que l'absence d'engagement contractuel est la faille majeure de l'offre.


## Résultats de l'analyse

- Le taux de churn global est de **26,5 %** sur les 7 043 clients analysés.
- Les contrats mensuels sans engagement concentrent la quasi-totalité des départs, ce qui désigne la formule contractuelle comme principal point faible de l'offre.
- Les clients équipés en fibre optique partent davantage que ceux en ADSL, ce qui peut indiquer un problème de qualité de service ou de tarification sur ce segment.
- Les clients qui partent ne sont pas des petits comptes : leur facture moyenne est supérieure à celle des clients fidèles. Le churn touche donc directement les segments à forte valeur.

## Limites

- Les données sont américaines et la conversion en FCFA repose sur un taux fixe, sans tenir compte des tarifs ni du pouvoir d'achat locaux.
- L'analyse est descriptive : elle explique le churn mais ne le prédit pas. Un modèle de scoring (régression logistique, Random Forest) serait une suite naturelle.

## Outils utilisés

Python 3.11 (Pandas, NumPy, Matplotlib, Seaborn, SQLAlchemy) · PostgreSQL 16 (pgAdmin 4) · Power BI Desktop · Git / GitHub

## Auteur

MOUTCHI MARIAMNE— JUNIOR Data Analyst, Douala, Cameroun
[LinkedIn](https://github.com/mariamnebalbine) · [GitHub](https://github.com/mariamnebalbine)

![Dashboard Power BI](captures/visualisation.png)

## 🎯 Résultat en une phrase

** 26,5 % des clients ont quitté l'opérateur, et la quasi-totalité des départs provient des contrats mensuels sans engagement. Le manque à gagner est chiffré en FCFA et une liste de clients VIP à relancer est livrée aux équipes marketing.