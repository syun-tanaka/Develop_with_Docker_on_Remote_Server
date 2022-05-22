FROM nvcr.io/nvidia/pytorch:22.03-py3

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Tokyo
RUN echo $TZ > /etc/timezone

ARG WORKDIR=/project
WORKDIR ${WORKDIR}

# change default shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN chsh -s /bin/bash

# Increase timeout for apt-get to 300 seconds
# hadolint ignore=DL3059
RUN /bin/echo -e "\n\
    Acquire::http::Timeout \"300\";\n\
    Acquire::ftp::Timeout \"300\";" >> /etc/apt/apt.conf.d/99timeout

# install hadolint
RUN wget -nv https://github.com/hadolint/hadolint/releases/download/v2.8.0/hadolint-Linux-x86_64 -O /usr/local/bin/hadolint \
    && chmod +x /usr/local/bin/hadolint

# install pyright
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && npm install --global pyright

ARG PACKAGES="tzdata screen libgl1-mesa-dev"

# For root user (Recommended for Rootless Docker)
ARG USER_NAME=root
ARG HOMEDIR=/$USER_NAME

# For non-root user (Recommended for (Normal) Docker)
# ARG USER_NAME=user
# ARG HOMEDIR=/home/$USER_NAME
# RUN useradd -s /bin/bash -m $USER_NAME \
#     # Add sudo support for non-root user
#     && mkdir -p /etc/sudoers.d \
#     && echo $USER_NAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USER_NAME \
#     && chmod 0440 /etc/sudoers.d/$USER_NAME
# ARG PACKAGES="${PACKAGES} sudo"

# Configure apt and install packages
RUN rm -rf /var/lib/apt/lists/* \
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends $PACKAGES\
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# terminal setting
RUN wget -nv https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -O $HOMEDIR/.git-completion.bash \
    && wget -nv https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh -O $HOMEDIR/.git-prompt.sh \
    && chown -R $USER_NAME:$USER_NAME $HOMEDIR \
    && chmod a+x $HOMEDIR/.git-completion.bash \
    && chmod a+x $HOMEDIR/.git-prompt.sh \
    && echo -e "\n\
    source $HOMEDIR/.git-completion.bash\n\
    source $HOMEDIR/.git-prompt.sh\n\
    export PS1='\\[\\e]0;\\u@\\h: \\w\\a\\]\${debian_chroot:+(\$debian_chroot)}\\[\\033[01;32m\\]\\u\\[\\033[00m\\]:\\[\\033[01;34m\\]\\w\\[\\033[00m\\]\\[\\033[1;30m\\]\$(__git_ps1)\\[\\033[0m\\] \\$ '\n\
    export HISTFILE=$WORKDIR/.docker_bash_history\n\
    export HISTSIZE=100000\n\
    export HISTFILESIZE=100000\n\
    export HISTCONTROL=ignoreboth\n\
    export PROMPT_COMMAND='history -a; history -c; history -r; $PROMPT_COMMAND'\n\
    shopt -u histappend\n\
    " >>  $HOMEDIR/.bashrc

# hadolint ignore=DL3013
RUN pip install --no-cache-dir -U pip setuptools

ENV DEBIAN_FRONTEND=
USER $USER_NAME
CMD ["/bin/bash"]
