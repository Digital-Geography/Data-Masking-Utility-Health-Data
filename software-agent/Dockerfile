# Sauberes, gut gepflegtes Basisimage
FROM jupyter/base-notebook:python-3.11

# Besseres Build-Logging & keine Cache-Leichen
ENV PIP_NO_CACHE_DIR=1 PIP_DISABLE_PIP_VERSION_CHECK=1

# Systempakete nur falls nötig (hier minimal)
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*
USER $NB_UID

# Python-Pakete
COPY requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

# Standard-Workdir (jovyan)
WORKDIR /home/jovyan/work

# Optional: Jupyter ohne Login-Token für lokale DEV (nicht für Prod!)
ENV JUPYTER_ENABLE_LAB=yes
