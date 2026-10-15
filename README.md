# GEO-MW : Open GEOspatial workflow to link urban Morphology variables and UrbanTALES Wind data
 
[![License: LGPL v3](https://img.shields.io/badge/License-LGPL_v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
 
This repository is associated with a research paper currently in preparation, which investigates the relationship between urban morphology and wind speed. The workflow processes geospatial data retrieved from UrbanTALES (Nazarian et al., 2025) and provides them with a spatial anchorage, a step referred to as georeferencing. Aeraulic and morphological variables are then computed on a 50 × 50 m regular grid with a horizontal resolution of 0.5 m (ranging from 0.25 m to 49.75 m). The full workflow is runnable from a Python script (`main.py`) or a notebook (`pipeline_process.ipynb`).

The data processing workflow involves converting 3D wind data stored in NetCDF files (`.nc` files) into raster layers, which are then georeferenced using manually collected ground control points.

## Table of contents
 
- [Data](#data)
- [Main goals of the repository](#main-goals-of-the-repository)
- [Installation](#installation)
  - [Requirements](#requirements)
  - [Downloading the data](#downloading-the-data)
  - [Folder layout and path configuration](#folder-layout-and-path-configuration)
  - [Cloning the repository and installing the dependencies](#cloning-the-repository-and-installing-the-dependencies)
- [Usage](#usage)
- [Repository structure](#repository-structure)
- [License](#license)
- [Acknowledgments](#acknowledgments)
- [Authors](#authors)

## Data 

This workflow relies on two data sources: the UrbanTALES dataset and the OpenStreetMap (OSM) database.

UrbanTALES (Nazarian et al., 2025) is a freely available dataset of large-eddy simulations (LES) of wind flow in urban environments, which allows a detailed analysis of airflow patterns within the urban fabric. These data do not include real-world geographical coordinates.

OpenStreetMap (OSM; Coast, 2004) is a crowdsourced geospatial database of volunteered geographic information.

## Main goals of the repository 

This repository provides code to:

-   georeference UrbanTALES wind and topography data, from a local coordinate-free reference to the local UTM zone of each scenario;
-   aggregate the data on a 50 × 50 m regular grid;
-   compute aeraulic variables from the georeferenced UrbanTALES data;
-   compute morphological variables from OSM using GeoClimate (Bocher et al., 2021).


## Installation
 
### Requirements
 


-   Python $\geq$ 3.10 (developed with Python 3.12)
-   Java ≥ 8 (GeoClimate relies on Java, so a reasonably recent Java installation is required on your computer)
-   Groovy \>= 4.0.29

### Downloading the data

The UrbanTALES data are available at <https://urbantales.vercel.app/> (select "realistic cases"). The following files are required:

-   the `topo` files, which contain building height information;
-   the `png` files, which show the building footprints.

To our knowledge, the 3D wind files are not yet available on the website.

Place the UrbanTALES data in the folders described in the next section. Then download the `data` folder of this repository and:

-   copy the content of `data/gdal_commands` into `Data/GeoReference/Georef_input`;
-   copy `metadata.csv` into `Data/UrbanTALES`.

The folder `0_expl_tmp` provides an example of the temporary files generated during the georeferencing process.

### Folder layout and path configuration

The data are stored outside the repository, with the following layout:

```
Projects/
├── Data/                          # data (outside the repository)
│   ├── UrbanTALES/
│   │   ├── 3Dwind/                # 3D wind data (NetCDF)
│   │   ├── png/                   # png files downloaded from UrbanTALES
│   │   ├── topo/                  # topo files downloaded from UrbanTALES
│   │   └── metadata.csv           # metadata file given in the repository
│   ├── Extracting_buildings/      # the georeferenced topo rasters (files are created during processing)
│   └── GeoReference/
│       ├── Georef_input/          # inputs for georeferencing (place here the content of data/gdal_commands)
│       └── Georef_output/         # outputs for georeferencing (where the outputs of this workflow are stored)
├── Outputs/                       # outputs (created automatically but not used)
└── GEO_MW/                        # this repository
```


Paths are defined in a single place: `scripts/config.py`.

-   The **project root** is detected automatically from the location of the code.
-   The **data folder** is not in the repository. By default, the code looks for it in `../Data` (next to the repository)

To place the data elsewhere, set an environment variable:

``` bash
export URBANTALES_DATA=/path/to/Data
export URBANTALES_OUTPUT=/path/to/Outputs   # optional
```

To make it permanent, add these lines to `~/.bashrc`.

From the code, paths are accessed as follows:

``` python
from scripts.config import DATA_DIR, OUTPUT_DIR, PATH_3D, GEOREF_INPUT, GEOREF_OUTPUT, check_paths
 
check_paths()   # checks that Data exists, with an explicit error message otherwise
```

### Cloning the repository and installing the dependencies

``` bash
git clone git@github.com:Lenaig-LG/georef-UrbanTALES.git
cd georef-UrbanTALES
 
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Usage

Always run from the repository root, with the virtual environment activated.

-   `main.py`: runs the processing for all scenarios listed in `scripts.scenario.py`
-   `pipeline_process.ipynb`: runs the processing pipeline one scenario at a time, with details on the steps and plots.

**Python script:**

``` bash
python main.py
```

**Notebook:**

``` bash
jupyter notebook pipeline_process.ipynb
```

The first cell of the notebook detects the repository root and sets up imports and paths.

## Repository structure

```         
GEO_MW/
├── scripts/                     
│   ├── config.py                # Project paths (configurable)
│   ├── final_grid.py            # Final grid
│   └── functions.py             # Utility functions
│   ├── geo_grid.py              # Grid of geographical variables
│   ├── grid.py                  # Grids (base class)
│   ├── metadata.py              # Metadata
│   ├── netcdf.py                # NetCDF reading 
│   ├── raster.py                # Raster handling
│   ├── scenario.py              # Listing of all scenarios used
│   ├── topo_raster_georef.py    # Georeferenced building height raster 
│   ├── topo_raster.py           # Building height raster 
│   ├── wind_grid.py             # Grid of wind variables
├── groovy/                      
│   ├── extractBuilding.groovy      # Extracting building from OSM
│   ├── geoClimateIndicators.groovy # Compute geo indicators on grid
│   └── WindDataGDALCommands.groovy # Georeferencing
├── main.py                      # Run the pipeline (on all scenarios)
├── pipeline_process.ipynb       # Run the pipeline (on one scenario at a time)
├── LICENSE                      # LGPL-3.0 text         
└── README.md
```

## License 

This project is distributed under the **GNU Lesser General Public License v3.0 or later** (`LGPL-3.0-or-later`).

The full text can be found in the [`LICENCE`](LICENCE) file (LGPL-3.0).

Copyright (C) 2026 Lenaig Le Grognec

## Acknowledgments 

We would like to thank the authors of UrbanTALES for generously sharing their 3D airflow data with us.

## Authors 

- Lenaig Le Grognec ([@Lenaig-LG](https://github.com/Lenaig-LG)), Lab-STICC, Université de Bretagne Sud
- Baptiste Alglave, Lab-STICC, Université de Bretagne Sud
- Erwan Bocher, CNRS, Lab-STICC
