name := laboro
tag := init
layer := init
eval := ""
isBuild := false
dns_cli := ../wireguard/bin/dns-cli

.PHONY : new-%
new-% : wg-svr docker-env
	$(dns_cli) add $(lastword $(subst :, , $*))
	$(dns_cli) key $*
	$(MAKE) create-ctl isBuild=false tag=$(firstword $(subst :, , $*)) name=$(lastword $(subst :, , $*))

.PHONY : run-%
run-% : wg-svr xorg-forward docker-env
	$(dns_cli) key $*
	lima nerdctl container start -a $*

.PHONY : run-%
del-% :
	$(dns_cli) del $*
	lima nerdctl container rm $*

.PHONY : %-image
%-image : docker-env images/%-image
	@

images/emqx-light-image : layers/emqx-light images/emqx-image
	$(MAKE) create-image-emqx-light tag=emqx

images/emqx-full-image : layers/emqx-full images/emqx-image
	$(MAKE) create-image-emqx-full tag=emqx

images/emqx-image : layers/emqx images/erlang-image
	$(MAKE) create-image-emqx tag=erlang

images/erlang-image : layers/erlang images/base-image
	$(MAKE) create-image-erlang tag=base

images/base-image : layers/base images/init-image
	$(MAKE) create-image-base tag=init

images/init-image: .ssh etc Dockerfile bin
	lima nerdctl build -t laborbenko:init .
	@touch $@

create-image-% :
	@echo "building the image: $*"
	rm -rf /tmp/lima/laborbenko/etc
	mkdir -p /tmp/lima/laborbenko/etc
#	cp -r etc/* /tmp/lima/laborbenko/etc/
	cp -r layers/* /tmp/lima/laborbenko/etc/
	$(MAKE) create-ctl name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true tag=$(tag)
	lima nerdctl commit laborbenko-$*-image laborbenko:$*
	lima nerdctl container rm laborbenko-$*-image
	@touch images/$*-image

.PHONY : create-ctl
create-ctl :
	lima nerdctl run -it --privileged --ulimit nofile=1024000:1024000 \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--name $(name) --hostname $(name) laborbenko:$(tag) -l $(layer) -e $(eval) -b $(isBuild)

.ssh :
	cp -r ~/.ssh .ssh

wireguard :
	cp -r ../wireguard wireguard

bin : entrypoint wireguard
	mkdir -p bin
	cp entrypoint bin/
	cp wireguard/bin/* bin/

.PHONY : docker-env
docker-env : lima docker-network docker-volume

.PHONY :
lima :
	@if [ "Running" != "$$(limactl list -f {{.Status}})" ]; then limactl start default; fi

RESULT := $(shell lima nerdctl network ls --format '{{.Name}}' | grep laboro)
.PHONY : docker-network
docker-network :
ifeq ($(RESULT),)
	lima nerdctl network create laboro
endif

RESULT := $(shell lima nerdctl volume ls --format '{{.Name}}' | grep laboro)
.PHONY : docker-volume
docker-volume :
ifeq ($(RESULT),)
	lima nerdctl volume create laboro
endif

.PHONY : wg-svr
wg-svr :
	@if [ $$(ps aux | grep '[d]ns-svr'|wc -l) -le 0 ]; \
		then bash -c "sudo nohup ./../wireguard/start-svr &"; fi

.PHONY : xorg-forward
xorg-forward :
	[ -z "$(lima nerdctl container ls --format '{{.Names}}' | grep xorg-forward)" ] \
		&& cd ../xorg-forward && $(MAKE) run

.PHONY : clean
clean :
	rm -rf wireguard
	rm -rf .ssh
	rm -rf /tmp/lima/laborbenko/etc
	rm -rf bin

.PHONY : clean-al
clean-all : clean
	rm -rf images/*
