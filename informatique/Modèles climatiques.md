---
schema_version: 1
uid: "01M02JG1VE294J3B6MB8G0AWKF"
titre: "Modèles climatiques"
aliases:
  - "CMIP"
  - "Modélisation du climat"
  - "Calcul scientifique et climat"
type: fiche
statut: actif
para: ressource
domaines:
  - enseignement
themes:
  - informatique
  - calcul-scientifique
  - climat
  - simulation
  - python
resume: "Fiche sur les modèles climatiques vus de l'informatique (2026) : chronologie corrigée des modèles et des machines (ENIAC, Manabe, Cray-1, exascale), fonctionnement d'un modèle de climat, projets d'intercomparaison CMIP6 et CMIP7 (Assessment Fast Track pour le GIEC AR7), modèles d'apprentissage automatique (AIFS opérationnel à l'ECMWF), accès aux données CMIP en Python avec xarray et intake-esm (exemple exécuté), et repères sur la recherche française."
auteurs:
  - "Michaël Launay"
langue: fr
date_creation: 2024-01-20
date_modification: 2026-08-29
confidentialite: publique
publication:
  - notes-publiques
rag: true
metadata_verifiees: false
---
# Modèles climatiques

> [!abstract] Objectif
> Donner des repères exacts sur ce qu'est un modèle de climat, sur la façon dont l'informatique — supercalculateurs, langages, formats, apprentissage automatique — a fait progresser la simulation du climat depuis 1950, sur l'organisation des projets d'intercomparaison CMIP6 et CMIP7, et montrer comment un informaticien accède aux données de ces modèles avec Python.

> [!info] État de la fiche
> Réécrite le 29 août 2026. La version de 2024 comportait des erreurs de faits (le Cray-1 date de 1976, non de 1972 ; AM4 est un modèle du GFDL et non du NCAR ; EC-Earth est un consortium européen et non un modèle de l'ECMWF ; MIROC est japonais, l'ICRISAT est un institut agronomique indien ; Copernicus date de 2014). L'exemple Python du chapitre 6 a été exécuté avec xarray 2026.7.

Voir aussi : [[Python]], [[Pandas]], [[Numpy]], [[Machine Learning]].

# 1. Ce qu'est un modèle de climat

Un **modèle de climat** résout numériquement les équations de la mécanique des fluides, de la thermodynamique et du transfert radiatif sur une grille couvrant l'atmosphère, l'océan, la glace et les surfaces continentales. Chaque processus trop petit pour la grille (nuages, convection, turbulence) est représenté par une **paramétrisation**. Un **modèle du système Terre** (ESM) ajoute le cycle du carbone, la chimie atmosphérique, les aérosols et la végétation.

Trois familles :

- **modèles de circulation générale** (GCM) et modèles du système Terre : globaux, résolution de 25 à 100 km, siècles simulés ;
- **modèles régionaux** (CORDEX) : quelques kilomètres sur un domaine limité, forcés aux frontières par un modèle global ;
- **modèles de prévision du temps** : mêmes équations, échéances de quelques jours, assimilation d'observations ; le climat est la statistique du temps, pas sa prévision.

Un modèle ne « prédit » pas une date : il produit des **projections** conditionnées à des scénarios d'émissions, et son crédit vient de sa capacité à reproduire le climat passé et les grandes structures observées.

# 2. Chronologie : modèles et machines

| Année | Modèle ou projet | Ce que l'informatique y a changé |
|---|---|---|
| 1950 | première prévision numérique du temps (Charney, Fjørtoft, von Neumann) sur l'**ENIAC** | 24 heures de calcul pour 24 heures de prévision |
| 1956 | premier modèle de circulation générale (Norman Phillips) | quelques semaines de climat simulé sur un IAS |
| 1967 | Manabe et Wetherald : première estimation crédible de la sensibilité au CO₂ (prix Nobel 2021) | modèle unidimensionnel de transfert radiatif |
| 1969 | modèle de bilan d'énergie de Budyko (et de Sellers) | rétroaction de l'albédo des glaces |
| 1975 | Manabe et Wetherald : premier GCM à doublement de CO₂ | GCM tridimensionnel au GFDL |
| 1976 | **Cray-1** : le supercalculateur vectoriel des centres météo et climat pendant vingt ans | 160 mégaflops |
| 1979 | rapport Charney (sensibilité climatique 1,5 à 4,5 °C) ; ECMWF émet ses premières prévisions | la fourchette tient toujours |
| 1988 | Hansen devant le Sénat américain, avec le GCM du GISS | trois scénarios, projections vérifiées depuis |
| 1990-2021 | rapports d'évaluation du GIEC, du premier (AR1) au sixième (AR6) | chaque cycle s'appuie sur une génération de CMIP |
| 1995 | naissance de **CMIP** (programme mondial de recherche sur le climat) | protocoles communs d'expériences et de sorties |
| 2005-2015 | grappes de calcul massivement parallèles ; format **NetCDF** et conventions **CF** | mêmes fichiers pour tous les modèles |
| 2014 | programme européen **Copernicus** ; service Copernicus Changement climatique (C3S) en 2015, réanalyse **ERA5** | données ouvertes à l'échelle mondiale |
| 2016-2022 | **CMIP6** : une quarantaine de centres, environ cent modèles, 20 pétaoctets de sorties | archive fédérée ESGF |
| 2023-2025 | premiers modèles de prévision par **apprentissage automatique** (GraphCast, Pangu-Weather, FourCastNet) ; **AIFS** de l'ECMWF opérationnel le 25 février 2025 | dix jours de prévision en quelques minutes sur GPU |
| 2024-2026 | **CMIP7** ; jumeau numérique **Destination Earth** ; modèles kilométriques ; **JUPITER**, premier calculateur exaflopique européen (Jülich) | simulations à 1-5 km, exascale |

# 3. CMIP6 et CMIP7

Le **Coupled Model Intercomparison Project** définit des expériences standard que tous les centres exécutent avec leur modèle, et un format de sortie commun : c'est ce qui permet de comparer les modèles, de constituer des ensembles et d'alimenter les rapports du GIEC.

**CMIP6** (expériences de 2016 à 2022, base de l'AR6 en 2021) : expériences de base **DECK** (contrôle préindustriel, historique, +1 % de CO₂ par an, quadruplement abrupt), scénarios **SSP** (SSP1-2.6, SSP2-4.5, SSP5-8.5…), 23 projets d'intercomparaison thématiques. Quelques modèles, avec leurs institutions réelles :

| Modèle | Institution |
|---|---|
| **IPSL-CM6A-LR** | Institut Pierre-Simon Laplace (France) |
| **CNRM-CM6-1** et **CNRM-ESM2-1** | Météo-France et CERFACS (France) |
| **EC-Earth3** | consortium européen d'une trentaine d'instituts, fondé sur le code de l'ECMWF |
| **CESM2** | NCAR (États-Unis) |
| **GFDL-CM4** et **GFDL-ESM4** (atmosphère AM4) | GFDL, NOAA (États-Unis) |
| **UKESM1** | Met Office et universités britanniques |
| **MIROC6** | JAMSTEC, Université de Tokyo et NIES (Japon) |
| **CanESM5** | Centre canadien de modélisation et d'analyse climatiques |
| **MPI-ESM1-2** | Institut Max-Planck de météorologie (Allemagne) |

**CMIP7** (protocole publié en 2025, expériences en cours) devient un programme continu. Le **Assessment Fast Track** est un sous-ensemble d'expériences à livrer rapidement pour l'AR7 du GIEC et pour les services climatiques : simulation historique étendue jusqu'en 2021 (puis 2025), expériences **pilotées par les émissions de CO₂** plutôt que par les concentrations, nouveaux scénarios ScenarioMIP (de « très bas » à « haut »), forçages historiques mis à jour, et un cadre d'évaluation rapide des simulations dès leur publication. Les premières sorties arrivent sur l'archive ESGF en 2026 ; les synthèses multi-modèles prendront, comme à chaque génération, plusieurs années.

# 4. L'informatique du climat en 2026

- **Machines** : les centres de calcul climatique sont passés des vecteurs Cray aux grappes de CPU puis aux GPU ; l'exascale (JUPITER en Europe, Frontier et El Capitan aux États-Unis) rend possibles les simulations globales à quelques kilomètres, où la convection n'est plus paramétrée.
- **Codes** : Fortran reste dominant dans les cœurs dynamiques (IFS, ICON, CESM, LMDZ), avec des portages GPU (ICON, IFS) ; Python domine l'analyse ; Julia et les DSL (GT4Py, PSyclone) montent pour les nouveaux codes.
- **Données** : **NetCDF** avec les conventions **CF**, **Zarr** pour le stockage objet dans le nuage, catalogues **intake-esm**, archive fédérée **ESGF** ; les sorties CMIP6 sont aussi disponibles en Zarr sur Google Cloud et AWS.
- **Apprentissage automatique** : modèles de prévision entraînés sur ERA5 (AIFS, GraphCast, Pangu, Aurora, NeuralGCM), émulateurs et paramétrisations apprises ; en climat, ils complètent les modèles physiques (ensembles, descente d'échelle) sans les remplacer, faute de pouvoir extrapoler à des états jamais observés.
- **Jumeaux numériques** : Destination Earth (Union européenne, ECMWF, ESA, EUMETSAT) combine simulations kilométriques, observations et apprentissage pour l'adaptation.

# 5. Recherche et formation en France

L'IPSL (Paris-Saclay), le CNRM (Météo-France, Toulouse), le CERFACS, le LSCE et les laboratoires du CNRS-INSU portent les modèles français ; le GENCI et l'IDRIS fournissent le calcul. L'INSU recense les acteurs et les formations : <https://www.insu.cnrs.fr/fr/climat>. Le site **ClimatHD** (Météo-France) et le portail **DRIAS** donnent accès aux projections régionalisées pour la France ; le service **Climate ADAPT** et le C3S aux données européennes.

# 6. Accéder aux données CMIP en Python

La chaîne standard : **xarray** (tableaux étiquetés, lecture NetCDF et Zarr), **dask** (calcul parallèle hors mémoire), **intake-esm** (catalogues CMIP), **cartopy** et **matplotlib** (cartes), **ESMValTool** ou **xclim** (diagnostics et indices climatiques).

```bash
python -m pip install xarray netCDF4 cftime dask intake-esm gcsfs xclim cartopy
```

Exemple exécuté sur un jeu de données synthétique au format CMIP (variable `tas`, température de l'air en surface, calendrier CF) : moyenne globale pondérée par la surface des mailles, moyennes annuelles, anomalie.

```python
"""Moyenne globale pondérée par la surface d'un champ de température, avec xarray."""
import numpy as np
import xarray as xr

temps = xr.date_range("1850-01-01", "1859-12-01", freq="MS", use_cftime=True)
lat = np.arange(-89, 90, 2.0)
lon = np.arange(1, 360, 2.0)
tendance = np.linspace(0, 0.3, temps.size)[:, None, None]                     # +0,3 K sur la période
champ = 288 - 40 * np.abs(np.sin(np.deg2rad(lat)))[None, :, None] + tendance + np.zeros((1, 1, lon.size))
ds = xr.Dataset(
    {"tas": (("time", "lat", "lon"), champ.astype("float32"), {"units": "K", "long_name": "Near-Surface Air Temperature"})},
    coords={"time": temps, "lat": lat, "lon": lon},
)

# les mailles rétrécissent vers les pôles : pondérer par cos(latitude) avant de moyenner
poids = np.cos(np.deg2rad(ds.lat))
poids.name = "weights"
moyenne_globale = ds.tas.weighted(poids).mean(("lat", "lon"))
annuelle = moyenne_globale.groupby("time.year").mean()
anomalie = annuelle - annuelle.sel(year=slice(1850, 1854)).mean()
print(anomalie.sel(year=1859).item())        # 0.212 K
ds.to_netcdf("tas_synthetique.nc")
```

Avec de vraies données, seule la lecture change : un catalogue intake-esm des sorties CMIP6 hébergées sur Google Cloud sélectionne un modèle, une expérience et une variable, et renvoie le même type d'objet.

```python
import intake

catalogue = intake.open_esm_datastore("https://storage.googleapis.com/cmip6/pangeo-cmip6.json")
selection = catalogue.search(source_id="IPSL-CM6A-LR", experiment_id="historical",
                             variable_id="tas", table_id="Amon", member_id="r1i1p1f1")
jeux = selection.to_dataset_dict(xarray_open_kwargs={"consolidated": True})
ds = next(iter(jeux.values()))            # xarray.Dataset, lu à la demande depuis Zarr
```

Points d'attention : les calendriers non grégoriens (`noleap`, `360_day`) imposent `cftime` ; les moyennes doivent être pondérées par la surface ; les grilles diffèrent d'un modèle à l'autre (regridding avec `xesmf`) ; les identifiants CMIP (`source_id`, `experiment_id`, `variant_label`) sont normalisés et documentés par le vocabulaire contrôlé du projet.

# 7. Sources

- WCRP, CMIP7 et Assessment Fast Track : <https://wcrp-cmip.org/cmip-phases/cmip7/> ; Dunne et al., *An evolving CMIP7 and Fast Track…*, Geoscientific Model Development, 2025 : <https://gmd.copernicus.org/articles/18/6671/2025/>
- Carbon Brief, « How CMIP7 will shape the next wave of climate science » (mai 2026) : <https://www.carbonbrief.org/guest-post-how-cmip7-will-shape-the-next-wave-of-climate-science>
- ECMWF, AIFS opérationnel (février 2025) : <https://www.ecmwf.int/en/forecasts/documentation-and-support/aifs>
- GIEC, sixième rapport d'évaluation : <https://www.ipcc.ch/report/ar6/wg1/>
- Manabe et Wetherald (1967), prix Nobel de physique 2021 : <https://www.nobelprize.org/prizes/physics/2021/summary/>
- Archive ESGF : <https://esgf.llnl.gov/> ; CMIP6 sur Google Cloud (Pangeo) : <https://console.cloud.google.com/marketplace/product/noaa-public/cmip6>
- xarray : <https://docs.xarray.dev/> ; intake-esm : <https://intake-esm.readthedocs.io/> ; xclim : <https://xclim.readthedocs.io/>
- CNRS-INSU, climat : <https://www.insu.cnrs.fr/fr/climat> ; DRIAS : <https://www.drias-climat.fr/> ; FAQ CMIP6 (Donnéesclimatiques.ca) : <https://donneesclimatiques.ca/ressource/faq-sur-le-cmip6/>
