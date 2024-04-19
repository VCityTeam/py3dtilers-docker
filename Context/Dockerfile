# References
# - on chosing your Python base container
#   https://pythonspeed.com/articles/base-image-python-docker-images/
#   https://stackoverflow.com/questions/52740556/python-3-7-docker-images

FROM python:3.10

RUN git clone https://github.com/VCityTeam/py3dtilers.git
WORKDIR py3dtilers
RUN pip install -e .
