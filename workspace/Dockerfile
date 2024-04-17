#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'workspace' base Image, visit its repository on Github
#    https://github.com/Laradock/workspace
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/workspace/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG LARADOCK_PHP_VERSION
FROM laradock/workspace:latest-${LARADOCK_PHP_VERSION}


ARG LARADOCK_PHP_VERSION

ENV DEBIAN_FRONTEND noninteractive

USER root

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} laradock && \
    useradd -l -u ${PUID} -g laradock -m laradock -G docker_env && \
    usermod -p "*" laradock -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      libzip-dev zip unzip \
      php${LARADOCK_PHP_VERSION}-zip \
      nasm && \
      php -m | grep -q 'zip'


ARG TZ='Europe/Moscow'
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/laradock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/laradock/aliases.sh && \
    chown laradock:laradock /home/laradock/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

USER laradock

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

USER root

# Add the composer.json
COPY ./composer.json /home/laradock/.composer/composer.json

# Make sure that ~/.composer belongs to laradock
RUN chown -R laradock:laradock /home/laradock/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}
RUN set -eux; \
      if [ "$COMPOSER_VERSION" = "1" ] || [ "$COMPOSER_VERSION" = "2" ] || [ "$COMPOSER_VERSION" = "2.2" ]; then \
          composer self-update --${COMPOSER_VERSION}; \
      else \
          composer self-update ${COMPOSER_VERSION}; \
      fi

USER laradock

ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

USER laradock

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/html/vendor/bin:$PATH"' >> ~/.bashrc


USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d


USER root

COPY ./ca-certificates/* /usr/local/share/ca-certificates/
RUN update-ca-certificates


COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub


ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    apt-get update \
    && apt-get install -yqq php${LARADOCK_PHP_VERSION}-redis \
;fi

#ARG INSTALL_SWOOLE=false
#
#RUN set -eux; \
#    if [ ${INSTALL_SWOOLE} = true ]; then \
#      # Install Php Swoole Extension
#      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
#        echo '' | pecl -q install swoole-2.0.10; \
#      elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
#        echo '' | pecl -q install swoole-4.3.5; \
#      elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "1" ]; then \
#        echo '' | pecl -q install swoole-4.5.11; \
#      elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
#        echo '' | pecl -q install swoole-4.8.12; \
#      else \
#        echo '' | pecl -q install swoole; \
#      fi; \
#      echo "extension=swoole.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini; \
#      ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/swoole.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-swoole.ini; \
#      php -m | grep -q 'swoole'; \
#    fi



ARG INSTALL_XLSWRITER=false
RUN set -eux; \
    if [ ${INSTALL_XLSWRITER} = true ]; then \
      # Install Php xlswriter Extension
      if [ $(php -r "echo PHP_MAJOR_VERSION;") != "5" ]; then \
        echo '' | pecl -q install xlswriter && \
        echo "extension=xlswriter.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/xlswriter.ini && \
        ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/xlswriter.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xlswriter.ini && \
        php -m | grep -q 'xlswriter'; \
      else \
        echo "PHP Extension for xlswriter is not supported for PHP 5.0"; \
      fi \
    ;fi



ARG INSTALL_TAINT=false

RUN if [ "${INSTALL_TAINT}" = true ]; then \
    # Install Php TAINT Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
      pecl install taint && \
      echo "extension=taint.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/taint.ini && \
      ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/taint.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-taint.ini && \
      php -m | grep -q 'taint'; \
    fi \
;fi


ARG INSTALL_LIBPNG=false

RUN if [ ${INSTALL_LIBPNG} = true ]; then \
    apt-get -yqq install libpng16-16 \
;fi



USER laradock

###########################################################################
# Node / NVM:
###########################################################################

# Check if NVM needs to be installed
ARG NODE_VERSION=node
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NODE=false
ARG INSTALL_NPM_GULP=false
ARG INSTALL_NPM_BOWER=false
ARG INSTALL_NPM_VUE_CLI=false
ARG INSTALL_NPM_ANGULAR_CLI=false
ARG INSTALL_NPM_CHECK_UPDATES_CLI=false
ARG NPM_REGISTRY
ENV NPM_REGISTRY ${NPM_REGISTRY}
ARG NPM_FETCH_RETRIES
ENV NPM_FETCH_RETRIES ${NPM_FETCH_RETRIES}
ARG NPM_FETCH_RETRY_FACTOR
ENV NPM_FETCH_RETRY_FACTOR ${NPM_FETCH_RETRY_FACTOR}
ARG NPM_FETCH_RETRY_MINTIMEOUT
ENV NPM_FETCH_RETRY_MINTIMEOUT ${NPM_FETCH_RETRY_MINTIMEOUT}
ARG NPM_FETCH_RETRY_MAXTIMEOUT
ENV NPM_FETCH_RETRY_MAXTIMEOUT ${NPM_FETCH_RETRY_MAXTIMEOUT}
ENV NVM_DIR /home/laradock/.nvm
ARG NVM_NODEJS_ORG_MIRROR
ENV NVM_NODEJS_ORG_MIRROR ${NVM_NODEJS_ORG_MIRROR}

RUN if [ ${INSTALL_NODE} = true ]; then \
    mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash \
        && . $NVM_DIR/nvm.sh \
        && nvm install ${NODE_VERSION} \
        && nvm use ${NODE_VERSION} \
        && nvm alias ${NODE_VERSION} \
        && npm cache clear --force \
        && npm config set fetch-retries ${NPM_FETCH_RETRIES} \
        && npm config set fetch-retry-factor ${NPM_FETCH_RETRY_FACTOR} \
        && npm config set fetch-retry-mintimeout ${NPM_FETCH_RETRY_MINTIMEOUT} \
        && npm config set fetch-retry-maxtimeout ${NPM_FETCH_RETRY_MAXTIMEOUT} \
        && if [ ${NPM_REGISTRY} ]; then \
        npm config set registry ${NPM_REGISTRY} \
        ;fi \
        && if [ ${INSTALL_NPM_GULP} = true ]; then \
        npm install -g gulp \
        ;fi \
        && if [ ${INSTALL_NPM_BOWER} = true ]; then \
        npm install -g bower \
        ;fi \
        && if [ ${INSTALL_NPM_VUE_CLI} = true ]; then \
        npm install -g @vue/cli \
        ;fi \
        && if [ ${INSTALL_NPM_ANGULAR_CLI} = true ]; then \
        npm install -g @angular/cli \
        ;fi \
        && if [ ${INSTALL_NPM_CHECK_UPDATES_CLI} = true ]; then \
        npm install -g npm-check-updates \
        ;fi \
;fi

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

USER root

RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/laradock/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi


RUN if [ ${INSTALL_NODE} = true ]; then \
    find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/bower/bin/bower /usr/local/bin/bower && \
    ln -s $NODE_MODS_DIR/gulp/bin/gulp.js /usr/local/bin/gulp && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue /usr/local/bin/vue && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-init /usr/local/bin/vue-init && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-list /usr/local/bin/vue-list \
;fi

RUN if [ ${NPM_REGISTRY} ]; then \
    . ~/.bashrc && npm config set registry ${NPM_REGISTRY} \
;fi

COPY ./.npmrc /root/.npmrc
COPY ./.npmrc /home/laradock/.npmrc



USER root

ARG INSTALL_PNPM=false
ENV PNPM_HOME="/home/laradock/.local/share/pnpm"
ENV PATH $PATH:/home/laradock/.local/share/pnpm

RUN if [ ${INSTALL_PNPM} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export PNPM_HOME="/home/laradock/.local/share/pnpm"' >> ~/.bashrc && \
    echo 'export PATH="$PNPM_HOME:$PATH"' >> ~/.bashrc && \
    npx pnpm add -g pnpm \
;fi

USER laradock

ARG INSTALL_SUPERVISOR=false

RUN if [ ${INSTALL_SUPERVISOR} = true ]; then \
    if [ ${INSTALL_PYTHON} = true ]; then \
    python -m pip install --upgrade supervisor && \
    echo_supervisord_conf > /etc/supervisord.conf && \
    sed -i 's/\;\[include\]/\[include\]/g' /etc/supervisord.conf && \
    sed -i 's/\;files\s.*/files = supervisord.d\/*.conf/g' /etc/supervisord.conf \
  ;fi \
;fi


USER root

ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y libmagickwand-dev imagemagick && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
      apt-get install -y git && \
      cd /tmp && \
      if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
        git clone https://github.com/Imagick/imagick; \
      else \
        git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
      fi && \
      cd imagick && \
      phpize && \
      ./configure && \
      make && \
      make install && \
      rm -r /tmp/imagick; \
    else \
      pecl install imagick; \
    fi && \
    echo "extension=imagick.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/imagick.ini && \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/imagick.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-imagick.ini && \
    php -m | grep -q 'imagick' \
;fi



USER root

ARG INSTALL_YAML=false
ARG INSTALL_FFMPEG=false

RUN set -eux; \
  if [ ${INSTALL_YAML} = true ]; then \
    apt-get install -yqq libyaml-dev; \
    if   [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        echo '' | pecl install -a yaml-1.3.2; \
    elif [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
        echo '' | pecl install yaml-2.0.4; \
    else \
        echo '' | pecl install yaml; \
    fi; \
    echo "extension=yaml.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini; \
    ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/yaml.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/35-yaml.ini; \
  fi; \
  if [ ${INSTALL_FFMPEG} = true ]; then \
    apt-get -yqq install ffmpeg; \
  fi

###########################################################################
# BBC Audio Waveform Image Generator:
###########################################################################

USER root

ARG INSTALL_AUDIOWAVEFORM=false

RUN if [ ${INSTALL_AUDIOWAVEFORM} = true ]; then \
  apt-get -y install wget make cmake gcc g++ libmad0-dev libid3tag0-dev libsndfile1-dev libgd-dev libboost-filesystem-dev libboost-program-options-dev libboost-regex-dev \
  && cd /tmp \
  && git clone https://github.com/bbc/audiowaveform.git \
  && cd audiowaveform \
  && git clone --depth=1 https://github.com/google/googletest.git -b release-1.11.0 \
  && mkdir build \
  && cd build \
  && cmake .. \
  && make \
  && make install \
;fi

#####################################
# poppler-utils:
#####################################
USER root

ARG INSTALL_POPPLER_UTILS=false

RUN if [ ${INSTALL_POPPLER_UTILS} = true ]; then \
  apt-get -y install poppler-utils antiword \
;fi


#USER root
#
#ARG INSTALL_WKHTMLTOPDF=false
#ARG WKHTMLTOPDF_VERSION=0.12.6-1
#
#RUN if [ ${INSTALL_WKHTMLTOPDF} = true ]; then \
#   ARCH=$(arch | sed s/aarch64/arm64/ | sed s/x86_64/amd64/) \
#   && apt-get install -y \
#   libxrender1 \
#   libfontconfig1 \
#   libx11-dev \
#   libjpeg62 \
#   libxtst6 \
#   fontconfig \
#   libjpeg-turbo8-dev \
#   xfonts-base \
#   xfonts-75dpi \
#   wget \
#   && wget "https://github.com/wkhtmltopdf/packaging/releases/download/${WKHTMLTOPDF_VERSION}/wkhtmltox_${WKHTMLTOPDF_VERSION}.focal_${ARCH}.deb" \
#   && dpkg -i "wkhtmltox_${WKHTMLTOPDF_VERSION}.focal_${ARCH}.deb" \
#   && apt -f install \
#;fi



RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."


USER root

ARG INSTALL_PHPDECIMAL=false

RUN if [ ${INSTALL_PHPDECIMAL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      echo 'decimal not support PHP 5.6'; \
    else \
      apt-get install -yqq libmpdec-dev \
      && pecl install decimal \
      && echo "extension=decimal.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini \
      && ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/decimal.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/30-decimal.ini \
      && php -m | grep -q 'decimal' \
    ;fi \
;fi

USER root

ARG INSTALL_ZMQ=false

RUN if [ ${INSTALL_ZMQ} = true ]; then \
    apt-get install --yes git libzmq3-dev \
    && git clone https://github.com/zeromq/php-zmq.git \
    && cd php-zmq \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && cd .. \
    && rm -fr php-zmq \
    && echo "extension=zmq.so" >> /etc/php/${LARADOCK_PHP_VERSION}/mods-available/zmq.ini \
    && ln -s /etc/php/${LARADOCK_PHP_VERSION}/mods-available/zmq.ini /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-zmq.ini \
;fi




USER root

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm -f /var/log/lastlog /var/log/faillog

WORKDIR /var/www/html
