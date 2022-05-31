FROM alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
	&& apk update \
	&& apk add xorg-server \
	&& apk add xf86-video-dummy \
	&& apk add xrdp \
	&& apk add x11vnc \
	&& apk add xrandr \
	&& adduser firest -D -H \
	&& echo -e "123456\n123456" | passwd firest
ADD etc /etc/xorg-forward
ENV DISPLAY=:0
EXPOSE 3389:3389
CMD /etc/xorg-forward/start
