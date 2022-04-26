# Dockerized py3dtilers application

## Building and running a py3dtilers application (with docker)

This container wraps [p3dtilers](https://github.com/VCityTeam/py3dtilers) (a Python tool and library).

Build the docker image with

```bash
docker build -t vcity/py3dtilers Context
```

Then run the container e.g. with

```bash
docker run --rm -t vcity/py3dtilers geojson-tiler --help
docker run --rm -t vcity/py3dtilers citygml-tiler --help
```

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
docker pull 3dcitydb/3dcitydb-pg:14-3.2-4.2.0
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
```bash
wget https://download.data.grandlyon.com/files/grandlyon/imagerie/2018/maquette/LYON_1ER_2018.zip
unzip LYON_1ER_2018.zip
cd LYON_1ER_2018
docker pull 3dcitydb/impexp:5.0.0
docker run --rm --network citydb-net --name 3dcitydb-impexp \
    -v $(pwd):/data \
    3dcitydb/impexp:5.0.0 import \
    -H 3dcitydb -d my_3dcitydb -u my_postgres -p my_dummy_password \
    *.gml
```

### 3. Run the citygml tiler of py3dtilers
```bash
echo "PG_HOST: 3dcitydb"               > Config.yml
echo "PG_PORT: 5432"                  >> Config.yml
echo "PG_NAME: my_3dcitydb"           >> Config.yml
echo "PG_USER: my_postgres"           >> Config.yml
echo "PG_PASSWORD: my_dummy_password" >> Config.yml
git clone git@github.com:VCityTeam/py3dtilers-docker.git
cd py3dtilers-docker
docker build -t vcity/py3dtilers Context
docker run --rm --network citydb-net -v $(pwd):/data -t vcity/py3dtilers citygml-tiler --db_config_path /data/Config.yml
```
