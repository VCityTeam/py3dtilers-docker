# References
# - on chosing your Python base container
#   https://pythonspeed.com/articles/base-image-python-docker-images/
#   https://stackoverflow.com/questions/52740556/python-3-7-docker-images

FROM python:3.7-buster

RUN set -ex \
  && git clone https://github.com/VCityTeam/py3dtilers.git \
  && mv py3dtilers/ py3dtilers.git \
  && cd py3dtilers.git \
  && pip install -e .

WORKDIR py3dtilers.git/
