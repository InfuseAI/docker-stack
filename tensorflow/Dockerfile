# Base Images
ARG ROOT_CONTAINER=nvcr.io/nvidia/tensorflow:22.11-tf2-py3
ARG BASE_CONTAINER=$ROOT_CONTAINER
FROM $BASE_CONTAINER

# Argument
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"
ARG PRIMEHUB_EXTENSION_VERSION="0.1.4"

# Change to root user and configure the setting.
USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
 && apt-get install -yq --no-install-recommends \
    wget \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one \
    tini \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8

ENV HOME=/home/$NB_USER \
    PATH=$HOME/.local/bin:$PATH

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
   # Add call to conda init script see https://stackoverflow.com/a/58081608/4413446
   echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    mkdir -p "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}"

# Primehub Application Bundles
RUN apt-get update && apt-get install -yq --no-install-recommends git openssh-server curl htop \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# USER $NB_UID
WORKDIR $HOME

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work && \
    fix-permissions /home/$NB_USER

# Install mlflow and other pypi packages.
COPY requirements.txt /etc/jupyter/
RUN pip install -r /etc/jupyter/requirements.txt

# Install NodeJS for PrimeHub Extension
RUN curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - \
 && apt-get install -y nodejs

# Jupyter Notebook Port: 8888
EXPOSE 8888

# Configure container startup
# ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Fix permissions on /etc/jupyter as root
USER root

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/

# Legacy for Jupyter Notebook Server, see: [#1205](https://github.com/jupyter/docker-stacks/issues/1205)
RUN fix-permissions /etc/jupyter/
RUN chmod -R a+rx /usr/local/bin/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_USER
