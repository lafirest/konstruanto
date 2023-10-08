name := laboro
tag := init
layer := init
eval := ""
isBuild := false
dns_cli := ../network/host-bin/dns-cli

.PHONY : new-%
new-%: name=$(lastword $(subst :, , $*)).test
new-%: tag=$(firstword $(subst :, , $*))
new-% : docker-env
	@echo "kreante laborbenko nomitan: $(name), bazu sur: $(tag)"
	@$(dns_cli) add $(name)
	@$(MAKE) --silent yakuake-title name=$(name)
	@$(MAKE) create-ctl isBuild=false tag=$(tag) name=$(name)

.PHONY : run-%
run-%: name=$(*).test
run-% : docker-env
	@$(dns_cli) add $(name)
	@$(MAKE) --silent yakuake-title name=$(name)
# nerdctl should use -a, but -ai for docker
	@docker container start -ai $(name)

.PHONY : del-%
del-%: name=$(*).test
del-% :
	@echo "forigant labrobenko:$(name)"
	@$(dns_cli) del $(name)
	@docker container rm $(name)

.PHONY : del-%
enter-%: name=$(*).test
enter-% :
	@echo "eniru la laborbenkon:$(name)"
	@docker container exec -it -u firest -w /home/firest $(name) bash

.PHONY : lsit
list :
	@echo "la laborbenkonj estas:"
	@docker container ls -a --filter ancestor=laborbenko:base --format '{{.Names}}'

.PHONY : fork-%
fork-%: name=$(lastword $(subst :, , $*))
fork-%: tag=$(firstword $(subst :, , $*))
fork-% :
	@echo "generante la idon..."
	@docker container commit $(tag).test laborbenko:$(tag)
	@$(MAKE) new-$(tag):$(name)

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
	rm -rf tmp/laborbenko/etc/
	mkdir -p tmp/laborbenko/etc/
	cp -r layers/* tmp/laborbenko/etc/
	$(MAKE) create-ctl name=laborbenko-$*-image mode=-it mount="-v $$(pwd)/tmp/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true tag=$(tag)
	docker commit laborbenko-$*-image laborbenko:$*
	docker container rm laborbenko-$*-image
	@touch images/$*-image

.PHONY : create-ctl
create-ctl :
	@docker run -it --privileged --ulimit nofile=1024000:1024000 \
			--network=laboro \
			--mount type=volume,source=laboro,destination=/mnt/laboro \
			--mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock \
			--mount type=bind,source=/usr/bin/docker,destination=/usr/bin/docker \
			--mount type=bind,source=/usr/libexec/docker/cli-plugins/docker-compose,destination=/usr/libexec/docker/cli-plugins/docker-compose \
			--add-host=host.docker.internal:host-gateway \
			$(mount) \
			--name $(name) --hostname $(name) laborbenko:$(tag) -l $(layer) -e $(eval) -b $(isBuild)

ssh : etc/ssh_config
	cp -r ~/.ssh ssh
	cp etc/ssh_config ssh/config

network :
	cp -r ../network network

cli-bin : entrypoint network
	mkdir -p cli-bin
	cp entrypoint cli-bin/
	cp network/bin/* cli-bin/

.PHONY : docker-env
docker-env : docker-network docker-volume

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

.PHONY : yakuake-title
yakuake-title:
	@qdbus org.kde.yakuake \
		/yakuake/tabs \
		setTabTitle \
		$$(qdbus org.kde.yakuake /yakuake/sessions activeSessionId) \
		${name}

.PHONY : clean
clean :
	rm -rf network
	rm -rf ssh
	rm -rf cli-bin
	rm -rf tmp

.PHONY : clean-al
clean-all : clean
	rm -rf images/*
