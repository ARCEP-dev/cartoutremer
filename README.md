
<!-- README.md is generated from README.Rmd. Please edit that file -->

# cartoutremer

<!-- badges: start -->

<!-- badges: end -->

Le package R `CartOutremer` permet de faciliter la cartographie des
territoires français d’Outre-Mer (DROM et COM) dans les outils R et
QGis.

Les territoires de France d’Outre-Mer inclus sont les suivants :

  - l’ensemble des DROM (Départements et Régions d’outre-mer)
      - Guadeloupe (971)
      - Martinique (972)
      - Guyane (973)
      - La Réunion (974)
      - Mayotte (976)
  - les COM (Collectivités d’outre-mer) suivantes :
      - Saint-Pierre-et-Miquelon (975)
      - Saint-Barthélémy (977)
      - Saint-Martin (978)

# Installation :

``` r
remotes::install_github("ARCEP-dev/cartoutremer")
```

# Exemple de traitement :

``` r

library(cartoutremer)

# import des contours des départements de France métropolitaine et des DROM en projection WGS1984 via l'API IGN 
library(httr)
api_ignadmin <- "https://wxs.ign.fr/administratif/geoportail/wfs"
url <- parse_url(api_ignadmin)
url$query <- list(service = "wfs",
                  request = "GetFeature",
                  srsName = "EPSG:4326",
                  typename = "ADMINEXPRESS-COG.LATEST:departement")

DEP_FRMETDROM <- build_url(url) %>% read_sf()

# transformation des DROM pour les afficher proches de la France métropolitaine
DEP_FRMETDROM.proches <-
  transfo_om(shape_origine = DEP_FRMETDROM %>%
                             # uniquement les DROM
                             filter(substr(INSEE_DEP,1,2) %in% "97"),
             var_departement = "INSEE_DEP",
             type_transfo = "v1")

# cartographie avec ggplot 
ggplot() +
  geom_sf(data = DEP_GEO_FRMET %>%
                 # uniquement les départements de France métropolitaine
                filter(!substr(INSEE_DEP,1,2) %in% "97") %>%
                # agrégation des DROM visuellement rapprochés
                rbind.data.frame(DEP_FRMETDROM.proches),
          color = "black")
```

# Ressources annexes :

  - Contours des communes de France métropolitaine et DROM en projection
    WGS1984 mis à disposition par l’Arcep sur
    [data.gouv.fr](https://www.data.gouv.fr/fr/datasets/contours-communes-france-administrative-format-admin-express-avec-arrondissements/)

  - Contours des communes des COM (Saint-Pierre-et-Miquelon,
    Saint-Barthélémy, Saint-Martin) mis à disposition par l’Arcep sur
    [data.gouv.fr](https://www.data.gouv.fr/fr/datasets/decoupage-administratif-des-com-st-martin-et-st-barthelemy-et-com-saint-pierre-et-miquelon-format-admin-express/)
