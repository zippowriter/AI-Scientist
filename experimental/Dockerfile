# Use Python 3.11 as the base image
FROM python:3.11-bullseye as base

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies including texlive-full
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget=1.21-1+deb11u1 \
    git=1:2.30.2-1+deb11u2 \
    build-essential=12.9 \
    libssl-dev=1.1.1w-0+deb11u1 \
    zlib1g-dev=1:1.2.11.dfsg-2+deb11u2 \
    libbz2-dev=1.0.8-4 \
    libreadline-dev=8.1-1 \
    libsqlite3-dev=3.34.1-3 \
    libncursesw5-dev=6.2+20201114-2+deb11u2 \
    xz-utils=5.2.5-2.1~deb11u1 \
    tk-dev=8.6.11+1 \
    libxml2-dev=2.9.10+dfsg-6.7+deb11u4 \
    libxmlsec1-dev=1.2.31-1 \
    libffi-dev=3.3-6 \
    liblzma-dev=5.2.5-2.1~deb11u1 \
    texlive-full=2020.20210202-3 \
    && rm -rf /var/lib/apt/lists/*

# Env vars
ENV PYTHONIOENCODING=utf-8 \
    LANG=C.UTF-8
# Python envs
ENV PYTHONUNBUFFERED=1 \
    # Disable writing bytecode (.pyc)
    PYTHONDONTWRITEBYTECODE=1 \
    # pip
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=on \
    PIP_ROOT_USER_ACTION=ignore \
    # Poetry
    POETRY_HOME=/opt/poetry \
    POETRY_VIRTUALENVS_CREATE=false \
    POETRY_NO_INTERACTION=1
ENV PATH="${POETRY_HOME}/bin:${PATH}"
RUN python -m pip install --upgrade pip setuptools wheel

# Set working directory
WORKDIR /app
# Install Poetry
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -sSL https://install.python-poetry.org | python -

FROM base as app
COPY ai_scientist ai_scientist
COPY data data
COPY templates templates
COPY pyproject.toml .
COPY poetry.lock .

RUN poetry --version && poetry install --only main

# Prepare NanoGPT data
RUN poetry run python data/enwik8/prepare.py && \
    poetry run python data/shakespeare_char/prepare.py && \
    poetry run python data/text8/prepare.py

# Set up baseline runs
RUN for dir in templates/*/; do \
    if [ -f "${dir}experiment.py" ]; then \
    cd "${dir}" || continue; \
    poetry run python experiment.py --out_dir run_0 && \
    poetry run python plot.py; \
    cd /app || exit; \
    fi \
    done

# Create entrypoint script
RUN printf '#!/bin/bash\n\
    poetry run python launch_scientist.py "$@"\n' > /app/entrypoint.sh && \
    chmod +x /app/entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/app/entrypoint.sh"]

# Set the default command to an empty array
CMD []
