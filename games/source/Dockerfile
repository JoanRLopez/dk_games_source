#
# Copyright (c) 2021 Matthew Penner
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
#

FROM        --platform=$TARGETOS/$TARGETARCH debian:bookworm-slim

LABEL       author="Matthew Penner" maintainer="matthew@pterodactyl.io"
LABEL       org.opencontainers.image.source="https://github.com/pterodactyl/yolks"
LABEL       org.opencontainers.image.licenses=MIT

ENV         DEBIAN_FRONTEND=noninteractive

RUN         dpkg --add-architecture i386 \
            && apt update \
            && apt upgrade -y \
            && apt install -y tar curl gcc g++ libc6-i386 lib32gcc-s1 libgcc-11-dev libgcc-12-dev \
            libcurl4-gnutls-dev:i386 libssl-dev:i386 libcurl4:i386 lib32tinfo6 libtinfo6:i386 \
            lib32z1 lib32stdc++6 libncurses5:i386 libcurl3-gnutls:i386 libsdl2-2.0-0:i386 \
            iproute2 gdb libsdl1.2debian libfontconfig1 telnet net-tools netcat-traditional \
            tzdata numactl wget tini ca-certificates \
            && apt install -y python3-pip \
            && useradd -m -d /home/container container

# Configuración para SteamCMD
ENV         STEAMCMD_URL="http://media.steampowered.com/installer/steamcmd_linux.tar.gz"
ENV         STEAMROOT=/opt/steamcmd
ENV         ASSETTO_SERVER_DIR=/home/container/assetto

RUN         mkdir -p ${STEAMROOT} \
            && cd ${STEAMROOT} \
            && curl -s ${STEAMCMD_URL} | tar -vxz \
            && ./steamcmd.sh +login anonymous +quit; exit 0

# Configuración adicional de entorno
ENV         LANG C.UTF-8
ENV         SERVER_MANAGER_DIR=/home/container/server-manager
ENV         SERVER_INSTALL_DIR=${SERVER_MANAGER_DIR}/assetto
ENV         USER=container HOME=/home/container

# Creación y configuración de directorios para el servidor
RUN         mkdir -p ${SERVER_MANAGER_DIR} ${SERVER_INSTALL_DIR} ${ASSETTO_SERVER_DIR} \
            && chown -R container:container ${SERVER_MANAGER_DIR} \
            && chown -R container:container ${SERVER_INSTALL_DIR} \
            && chown -R container:container ${ASSETTO_SERVER_DIR}

# Descarga e instalación del servidor de Assetto Corsa
RUN         ${STEAMROOT}/steamcmd.sh +login "anonymous" "anonymous" \
            +force_install_dir ${ASSETTO_SERVER_DIR} \
            +app_update 302550 validate \
            +quit

USER        container
WORKDIR     /home/container

STOPSIGNAL  SIGINT

# Copiar y configurar el script de entrada
COPY        --chown=container:container ./entrypoint.sh /entrypoint.sh
RUN         chmod +x /entrypoint.sh

# Instalar RCON CLI
RUN         cd /tmp/ \
            && curl -sSL https://github.com/gorcon/rcon-cli/releases/download/v0.10.3/rcon-0.10.3-amd64_linux.tar.gz > rcon.tar.gz \
            && tar xvf rcon.tar.gz \
            && mv rcon-0.10.3-amd64_linux/rcon /usr/local/bin/

# Solución temporal para libssl1.1 en sistemas x86_64
RUN         if [ "$(uname -m)" = "x86_64" ]; then \
                wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb && \
                dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb && \
                rm libssl1.1_1.1.0g-2ubuntu4_amd64.deb; \
            fi

# Entrada del contenedor
ENTRYPOINT  ["/usr/bin/tini", "-g", "--", "/entrypoint.sh"]
CMD         ["./server-manager"]
