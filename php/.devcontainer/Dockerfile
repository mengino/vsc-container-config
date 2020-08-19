#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM php:7.1.33-fpm-buster

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Use ustc source insteads of debian official source
RUN sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
    && sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list \
    && sed -i 's/http/https/g' /etc/apt/sources.list \
    # Configure apt and install packages
    && apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # install git iproute2, procps, lsb-release (useful for CLI installs)
    && apt-get -y install git openssh-client less iproute2 procps iproute2 lsb-release zlib1g-dev libpng-dev libjpeg-dev libfreetype6-dev zip locales zsh ffmpeg \
    # Set git EOL always = lf
    && git config --global core.eol lf \
    && git config --global core.autocrlf input \
    && git config --global credential.helper store \
    #
    # Install xdebug
    # && yes | pecl install xdebug \
    # && echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" > /usr/local/etc/php/conf.d/xdebug.ini \
    # && echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    # && echo "xdebug.remote_autostart=on" >> /usr/local/etc/php/conf.d/xdebug.ini \
    #
    # Install php extension
    && docker-php-ext-install bcmath pdo_mysql shmop sysvmsg sysvsem sysvshm sockets pcntl zip \
    && docker-php-source extract \
    && cd /usr/src/php/ext/gd \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && docker-php-ext-enable gd \
    #
    # Install redis extension
    && pecl install redis && docker-php-ext-enable redis \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Install composer
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    # Use aliyun source insteads of packigst official source
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
    #
    # Set locale language
    && echo en_US.UTF-8 UTF-8 > /etc/locale.gen \ && locale-gen \
    #
    # Install oh-my-zsh
    && sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" \
    && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && git clone https://github.com/bhilburn/powerlevel9k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel9k \
    && git clone https://github.com/powerline/powerline.git /usr/local/powerline \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog
