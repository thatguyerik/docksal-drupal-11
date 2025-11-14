FROM docksal/cli:php8.3-3

USER docker
SHELL ["/bin/bash", "-c"]

RUN \
    . $HOME/.profile && \
    nvm install 22.19.0 && \
    nvm alias default 22.19.0

USER root
