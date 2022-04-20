# Dockerized py3dtilers application

This container wraps [p3dtilers](https://github.com/VCityTeam/py3dtilers) (a Python tool and library).

Build the docker image with

```bash
docker build -t vcity:py3dtilers Context
```

Then run the container e.g. with

```bash
docker run --rm -t vcity:py3dtilers geojson-tiler --help
docker run --rm -t vcity/py3dtilers citygml-tiler --help
```

## Running the tiler with docker

References:
* [3D City Database using Docker](https://3dcitydb-docs.readthedocs.io/en/version-2022.0/3dcitydb/docker.html)
* [ExpeData shellscript version](https://github.com/VCityTeam/ExpeData-Workflows_testing/blob/master/ShellScriptCallingDocker/LaunchDataBaseSingleServer.sh.j2)
```bash
docker pull 3dcitydb/3dcitydb-pg:14-3.2-4.2.0
docker network create citydb-net
docker run --name 3dcitydb --network citydb-net -p 5432:5432 -d \
    -e POSTGRES_PASSWORD=my_dummy_password \
    -e SRID=3946 \
    -e HEIGHT_EPSG=espg:3946 \
    -e POSTGRES_DB=my_3dcitydb \  
    -e POSTGRES_USER=my_postgres \
    -e POSTGIS_SFCGAL=true \
  3dcitydb/3dcitydb-pg:14-3.2-4.2.0
PGPASSWORD=my_dummy_password psql -h localhost -p 5432 -U my_postgres -d my_3citydb -c "\dt"
```

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
    -H 3dcitydb -d my_3citydb -u my_postgres -p my_dummy_password \
    *.gml
```
