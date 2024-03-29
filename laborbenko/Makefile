name := laboro
tag := init
layer := init
eval := ""
isBuild := false
dns_cli := ../wireguard/bin/dns-cli

.PHONY : new-%
new-%: name=$(lastword $(subst :, , $*))
new-%: tag=$(firstword $(subst :, , $*))
new-% : wg-svr docker-env
	@echo "kreante laborbenko nomitan: $(name), bazu sur: $(tag)"
	@$(dns_cli) add $(name)
	@$(dns_cli) key $(name)
	@$(MAKE) create-ctl isBuild=false tag=$(tag) name=$(name)

.PHONY : run-%
run-% : wg-svr docker-env
	@$(dns_cli) key $*
	@docker container start -a $*

.PHONY : del-%
del-% :
	@echo "forigant labrobenko:$*"
	@$(dns_cli) del $*
	@docker container rm $*

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

images/init-image: ssh etc Dockerfile cli-bin
	docker build -t laborbenko:init .
	@touch $@

create-image-% :
	@echo "building the image: $*"
	rm -rf /tmp/lima/laborbenko/etc
	mkdir -p /tmp/lima/laborbenko/etc
#	cp -r etc/* /tmp/lima/laborbenko/etc/
	cp -r layers/* /tmp/lima/laborbenko/etc/
	$(MAKE) create-ctl name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true tag=$(tag)
	docker commit laborbenko-$*-image laborbenko:$*
	docker container rm laborbenko-$*-image
	@touch images/$*-image

.PHONY : create-ctl
create-ctl :
	@docker run -it --privileged --ulimit nofile=1024000:1024000 \
			--network=laboro \
			--mount type=volume,source=laboro,destination=/mnt/laboro \
			$(mount) \
			--name $(name) --hostname $(name) laborbenko:$(tag) -l $(layer) -e $(eval) -b $(isBuild)

ssh : etc/ssh_config
	cp -r ~/.ssh ssh
	cp etc/ssh_config ssh/config

wireguard :
	cp -r ../wireguard wireguard

cli-bin : entrypoint wireguard
	mkdir -p cli-bin
	cp entrypoint cli-bin/
	cp wireguard/bin/* cli-bin/

.PHONY : docker-env
docker-env : lima docker-network docker-volume

.PHONY :
lima :
	@./bin/start-lima

RESULT := $(shell docker network ls --format '{{.Name}}' | grep laboro)
.PHONY : docker-network
docker-network :
ifeq ($(RESULT),)
	docker network create laboro
endif

RESULT := $(shell docker volume ls --format '{{.Name}}' | grep laboro)
.PHONY : docker-volume
docker-volume :
ifeq ($(RESULT),)
	docker volume create laboro
endif

.PHONY : wg-svr
wg-svr :
	@./bin/start-wg-svr

.PHONY : clean
clean :
	rm -rf wireguard
	rm -rf ssh
	rm -rf /tmp/lima/laborbenko/etc
	rm -rf cli-bin

.PHONY : clean-al
clean-all : clean
	rm -rf images/*
