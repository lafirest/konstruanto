FROM debian:testing-slim
RUN passwd -d root \
    && useradd -m -p 19910306 firest \
    && mkdir -p /mnt/laboro \
    && chown firest:firest /mnt/laboro \
    && runuser - firest -c 'mkdir -p ~/.local/bin' \
    && echo "deb http://mirrors.ustc.edu.cn/debian testing main contrib non-free non-free-firmware" > /etc/apt/sources.list \
    && rm /etc/apt/sources.list.d/*
ENV http_proxy=http://host.docker.internal:20172 \
    https_proxy=http://host.docker.internal:20172 \
    CCACHE_DIR=/mnt/laboro/ccache \
    TZ=Asia/Shanghai
COPY --chown=firest:firest ssh /home/firest/.ssh
COPY --chown=firest:firest etc/gitconfig /home/firest/.gitconfig
COPY cli-bin/* /usr/bin/
ENTRYPOINT ["entrypoint"]
