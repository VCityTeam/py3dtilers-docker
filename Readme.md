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
