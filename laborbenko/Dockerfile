FROM debian:testing
RUN useradd -m -p 19910306 firest \
    && runuser - firest -c 'mkdir -p ~/.local/bin' \
    && echo "deb http://mirrors.ustc.edu.cn/debian testing main contrib non-free non-free-firmware" > /etc/apt/sources.list \
    && rm /etc/apt/sources.list.d/*
ENV DISPLAY=xorg-forward:0 \
    http_proxy=http://host.docker.internal:20172 \
    https_proxy=http://host.docker.internal:20172 \
    CCACHE_DIR=/mnt/laboro/ccache \
    CC="ccache gcc" \
    CXX="ccache g++"
COPY --chown=firest:firest .ssh /home/firest/.ssh
COPY wireguard/etc/wgcli.conf /etc/wireguard/wgcli.conf
COPY entrypoint /usr/bin/entrypoint
ENTRYPOINT ["entrypoint"]
