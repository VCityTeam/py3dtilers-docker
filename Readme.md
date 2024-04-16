# Dockerized py3dtilers application

## Building and running a py3dtilers application (with docker)

This container wraps [p3dtilers](https://github.com/VCityTeam/py3dtilers) (a Python tool and library).

Build the docker image without cloning this repository with

```bash
docker build -t vcity/py3dtilers https://github.com/VCityTeam/py3dtilers-docker.git -f Context/Dockerfile
```

Or you can clone this repository and then proceed with the building with
```bash
git clone https://github.com/VCityTeam/py3dtilers-docker.git
docker build -t vcity/py3dtilers Context
```

Then run the container e.g. with

```bash
docker run --rm -t vcity/py3dtilers geojson-tiler --help
docker run --rm -t vcity/py3dtilers citygml-tiler --help
```

## Usage example : GeoJson : 3D Tiles colored by height

Download the GeoJSON file available [here](https://raw.githubusercontent.com/VCityTeam/UD-Sample-data/master/GeoJSON/buildings_lyon1.geojson).

```bash
docker run --rm -v <absolute/path/to/folder>:/mnt/data/ -t vcity/py3dtilers geojson-tiler --path /mnt/data/buildings_lyon1.geojson --add_color HAUTEUR -o /mnt/data/buildings_colored_by_height
```

Where `<absolute/path/to/folder>` is the __absolute path__ to the folder containing the GeoJSON file.

The tileset will be created in a folder named `<absolute/path/to/folder>/buildings_colored_by_height`.



## Usage example: running the citygml-tiler (with docker)

Running the [citygml tiler of py3dtilers](https://github.com/VCityTeam/py3dtilers/blob/master/py3dtilers/CityTiler/README.md)
requires a [3DCity database](https://www.3dcitydb.org/3dcitydb/) loaded with imported `citygml` content.
The following set of commands (requiring a functionnal [docker client](https://en.wikipedia.org/wiki/Docker_(software)) 
illustrates a full citygml tiling process realized from the CLI: 

### 1. Start a 3DCity database
References:
* [3D City Database using Docker](https://3dcitydb-docs.readthedocs.io/en/version-2022.0/3dcitydb/docker.html)
* [ExpeData shellscript version](https://github.com/VCityTeam/ExpeData-Workflows_testing/blob/master/ShellScriptCallingDocker/LaunchDataBaseSingleServer.sh.j2)
```bash
docker network create citydb-net
docker run --rm -d --name 3dcitydb --network citydb-net -p 5432:5432 \
    -e POSTGRES_PASSWORD=my_dummy_password \
    -e SRID=3946 \
    -e HEIGHT_EPSG=espg:3946 \
    -e POSTGRES_DB=my_3dcitydb \
    -e POSTGRES_USER=my_postgres \
    -e POSTGIS_SFCGAL=true \
  3dcitydb/3dcitydb-pg:14-3.2-4.2.0
PGPASSWORD=my_dummy_password psql -h localhost -p 5432 -U my_postgres -d my_3dcitydb -c "\dt"
```

### 2. Download CityGML files and import them to 3DCity database
References:
* [Using the Importer/Exporter with Docker](https://3dcitydb-docs.readthedocs.io/en/version-2022.0/impexp/docker.html)
* [ExpeData shellscript version: importation](https://github.com/VCityTeam/ExpeData-Workflows_testing/blob/master/ShellScriptCallingDocker/DockerLoad3dCityDataBase.sh)

Download CityGML data to your local filesystem e.g.

```bash
wget https://download.data.grandlyon.com/files/grandlyon/imagerie/2018/maquette/LYON_1ER_2018.zip
unzip LYON_1ER_2018.zip
rm -f LYON_1ER_2018.zip
```

Import local CityGML files to 3DCity database

```bash
docker run --rm --network citydb-net --name 3dcitydb-impexp \
    -v $(pwd):/data \
    3dcitydb/impexp:5.0.0 import \
    -H 3dcitydb -d my_3dcitydb -u my_postgres -p my_dummy_password \
    LYON_1ER_2018/*.gml
```

### 3. Run the citygml tiler of py3dtilers

Create py3dTilers configuration file describing access to 3dCityDB occurence

```bash
echo "PG_HOST: 3dcitydb"               > Config.yml
echo "PG_PORT: 5432"                  >> Config.yml
echo "PG_NAME: my_3dcitydb"           >> Config.yml
echo "PG_USER: my_postgres"           >> Config.yml
echo "PG_PASSWORD: my_dummy_password" >> Config.yml
```

Build the `CityTiler` and run it against current state of 3dCityDB occurence

```bash
git clone git@github.com:VCityTeam/py3dtilers-docker.git
docker build -t vcity/py3dtilers py3dtilers-docker/Context
docker run --rm --network citydb-net -v $(pwd):/data -t vcity/py3dtilers citygml-tiler --db_config_path /data/Config.yml
```
