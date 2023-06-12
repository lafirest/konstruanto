FROM debian:testing
RUN useradd -m -p 19910306 firest \
    && echo "deb http://mirrors.ustc.edu.cn/debian testing main contrib non-free non-free-firmware" > /etc/apt/sources.list \
    && rm /etc/apt/sources.list.d/*
ENV DISPLAY=xorg-forward:0 \
    http_proxy=http://host.docker.internal:20172 \
    https_proxy=http://host.docker.internal:20172 
COPY .ssh /home/firest/.ssh
COPY wireguard/limacli.conf /etc/wireguard/limacli.conf
COPY etc  /etc/laborbenko
COPY entrypoint /usr/bin/entrypoint
ENTRYPOINT /usr/bin/entrypoint
