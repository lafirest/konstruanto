FROM alpine
# It is necessary to install the font, otherwise the i3 font will be extremely small
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
	&& apk update \
	&& apk add xorg-server \
	&& apk add xf86-video-dummy \
	&& apk add xrdp \
	&& apk add x11vnc \
	&& apk add xrandr \
	&& apk add i3wm \
    && apk add font-dejavu \ 
	&& adduser firest -D -H \
	&& echo -e "123456\n123456" | passwd firest
COPY etc /etc/xorg-forward
ENV DISPLAY=:0
EXPOSE 3389:3389
CMD /etc/xorg-forward/start
