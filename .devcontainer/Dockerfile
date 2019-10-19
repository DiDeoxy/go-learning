# debian buster
FROM debian:10.1

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

# update apt-get and install needed packages
RUN apt-get update --fix-missing \
    && apt-get install -y --no-install-recommends \
        sudo \
        wget \
        bzip2 \
        ca-certificates \
        curl \
        bash-completion \
        less \
        ssh \
        git \
        pandoc \
        libcairo2-dev \
        libopenblas-dev \
    # clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ARG USERNAME=maxh
# On Linux, replace with your actual UID, GID if not the default 1000
ARG USER_UID=1001
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && mkdir -p /home/$USERNAME/.vscode-server /home/$USERNAME/.vscode-server-insiders \
    && chown ${USER_UID}:${USER_GID} /home/$USERNAME/.vscode-server* \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Set the default user
USER $USERNAME
WORKDIR /home/$USERNAME

# setup ssh
RUN echo 'eval $(ssh-agent -s)' >> /home/$USERNAME/.bashrc \
    && echo 'ssh-add ~/.ssh/id_rsa' >> /home/$USERNAME/.bashrc

# install conda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-4.5.11-Linux-x86_64.sh -O /home/maxh/miniconda.sh \
    && /bin/bash /home/$USERNAME/miniconda.sh -b -p /home/$USERNAME/conda \
    && rm /home/$USERNAME/miniconda.sh \
    && /home/$USERNAME/conda/bin/conda clean -tipsy \
    && sudo ln -s /home/$USERNAME/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    && echo ". /home/$USERNAME/conda/etc/profile.d/conda.sh" >> /home/$USERNAME/.bashrc \
    && echo "conda activate base" >> /home/$USERNAME/.bashrc

# add conda to path
ENV PATH /home/$USERNAME/conda/bin:$PATH

# install R through conda
RUN conda update conda \
    && conda install -c conda-forge go
    
# install tini?
ENV TINI_VERSION v0.16.1
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/bin/tini
RUN sudo chmod +x /usr/bin/tini

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]