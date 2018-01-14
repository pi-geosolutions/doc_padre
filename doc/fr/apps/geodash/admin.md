#Instance principale Geodash

http://dev.padre2.pigeo.fr/geodash/dashboard/private/index.html

![image](https://cloud.githubusercontent.com/assets/1491924/25302580/54b4ec72-2741-11e7-87f9-76ac86d2bf00.png)


Colonne de gauche: Liste des indicateurs existants
Centre: Détail de l'indicateur sélectionné

## Créer un indicateur

Cliquer sur le bouton `Add new` sous la colonne des indicateurs.
![image](https://cloud.githubusercontent.com/assets/1491924/25302690/11643a9c-2744-11e7-958c-b3612b764653.png)

Un nouvel indicateur vide est créé. Un bandeau vert avec un message `Nouvel indicateur` apparaît pour vous informer que l'indicateur n'a pas été enregistré en base de donnée.

![image](https://cloud.githubusercontent.com/assets/1491924/25302693/2d547802-2744-11e7-8112-363c468d48d0.png)

Attention: si vous allez sur autre autre indicateur, toutes les données du nouvel indicateur créé seront perdus.
Cliquez sur le bouton vert `Sauvegarder` pour être sur de bien avoir enregistré votre nouvel indicateur.

Un indicateur est de 3 sections:
- informations
- sources de données
- représentation graphique

### Global
Les informations à saisir sont
- nom: un identifiant unique pour l'indicateur
- label: le titre de l'indicateur qui sera affichée dans le dashboard
- commentaire: descriptions complémentaires

### Sources de données
Une source de données représente l'accès aux donner permettant d'alimenter le graph.
En général, une séride de donnée sur le graph (courbe, colonnes) est alimentée par une source de données. Mais ce n'est pas toujours le cas, il se peut qu'une série soit calculée à partir de plusieurs sources de données combinées (pourcentage, range ..).

Voici les options des sources de données:
- `name`: identifie la datasource (n'a pas d'importance)
- `type`: source de type fichier ou base de données
- `path`: Répertoire sur le serveur ou se trouvent l'ensemble des fichiers qui constituent la source.
- `pattern`: expression permettant de sélectionner les fichiers cibles dans le répertoire
- `nrelative`: indique si la série a une année relative à une autre série, et non un pattern fixe
- `amount`: quantité de fichier à récupérer pour constituer la source. Si 0 ou -1, tous les fichiers qui matchent l'expression sont utilisés
- `merge with previous`: indique que cette source ne va pas constituer une série dans le graph, mais va être mergée avec une autre source pour ne constituer qu'une seule série. Les merges possibles sont pourcentage ou concaténation.
- basée sur une autre source: indique si le calcul de la source est basée sur les données d'une autre source par exemple pour le calcul de l'écart type ou autre.

#### Exemples de pattern
Mots clé dans les expressions
- `$year$` l'expression extrait l'année dans les noms de fichiers. L'année est pas un datetime, elle représente seulement une catégorie, le service renvoie alors la liste des catégories trouvées en plus des données.
Pour year, la dernière année est retirée des résultats pour ne pas traiter l'année en cours.
- `$month$` idem que `$year$`, c'est une catégorie, mais la dernière catégorie est conservée.
- `$dateYYmm$` date indique que le pattern est un datetime, la suite du pattern est un format de date type ISO, on peut avoir `YYYYMMdd` `YYYY` `YYYYMMddThh:mm:ss` etc...
- `----$dateMMdd$` `----` Indique que l'année est modifiable, les 4 - correspondent aux 4 chiffres de l'année. Couplé à la date, le pattern est transformé dynamiquement en `2016$dateMMdd$` par exemple, on peut ainsi extraire le datetime dynamiquement pour chaque fichier.

Exemples de patterns
```
mpe_$dateyyMMdd_HHmm$.png
RainCum----$dateMMdd$.tif
ESA$month$.tif
africa_rfe.$dateyyyyMMdd$.tif
```

Exemple de source de donnée simple.
On veut afficher un graphique colonnes avec la quantité de pluie ces 15 dernières années.

![image](https://cloud.githubusercontent.com/assets/1491924/25302790/b5328794-2746-11e7-8aa4-3d3122b4c833.png)

## Graphique
La configuration du graph est une configuration Highchart avec des fonctions helpers pour traiter des besoins particuliers.

voir
- examples: https://www.highcharts.com/demo
- doc: http://api.highcharts.com/highcharts

### Les helpers:
- monthInYearAxisLabelFormatter
- periodFormatter
- monthInYearAxisLabelPositioner

```json
    "xAxis": {
        "labels": {
            "formatter": "$eval:monthInYearAxisLabelFormatter"
        }
    },
```

```json
    "yAxis": {
        "tickInterval": 30.41,
        "tickPositioner": "$eval:monthInYearAxisLabelPositioner",
```

```json
    "tooltip": {
        "valueSuffix": "°C",
        "formatter": "$eval:periodFormatter",
        "backgroundColor": "white"
    }
```

### Regression linéaire
Un plugin permet de calculer automatiquement les regressions linéaires

```json
    "series": [
        {
            "name": "% jours de pluie",
            "zIndex": 0,
            "type": "column",
            "tooltip": {
                "valueSuffix": " %"
            },
            "regression": true ,
            "regressionSettings": {
                "type": "linear",
                "name": "Regression",
                "color":  "rgba(223, 83, 83, .9)",
                "enableMouseTracking": false,
                "hideInLegend": true
            }
        }
    ]
```