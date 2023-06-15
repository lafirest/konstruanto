qiname := laboro
tag := init
layer := init
ip := "10.255.0.2"
eval := ""
isBuild := false

.PHONY : run
run : docker-env
	lima nerdctl run -it --privileged \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--rm --name $(name) laborbenko:$(tag) -l $(layer) -i $(ip) -e $(eval) -b $(isBuild)

.PHONY : create
create :
	lima nerdctl run -it --privileged \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--name $(name) laborbenko:$(tag) -l $(layer) -i $(ip) -e $(eval) -b $(isBuild)

.PHONY : docker-env
docker-env : docker-network docker-volume

.PHONY : %-image
%-image : docker-env images/%-image
	@

images/emqx-image : images/erlang-image
	$(MAKE) create-image-emqx tag=erlang

images/erlang-image : images/base-image
	$(MAKE) create-image-erlang tag=base

images/base-image : images/init-image
	$(MAKE) create-image-base tag=init

create-image-% : ip := $(shell wireguard/bin/alloc_ip true)
create-image-% : etc/%
	@echo "building the image: $*"
	mkdir -p /tmp/lima/laborbenko
	rm -rf /tmp/lima/laborbenko/etc
	cp -r etc /tmp/lima/laborbenko/etc
	$(MAKE) create name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true ip=$(ip) tag=$(tag)
	lima nerdctl commit laborbenko-$*-image laborbenko:$*
	lima nerdctl container rm laborbenko-$*-image
	@touch images/$*-image

images/init-image: .ssh wireguard Dockerfile entrypoint
	lima nerdctl build -t laborbenko:init .
	@touch $@

.PHONY : %-image-test
%-image-test : images/%-image-test
	@

images/%-image-test :
	@echo "building the image: $*"
	mkdir -p /tmp/lima/laborbenko
	rm -rf /tmp/lima/laborbenko/etc
	cp -r etc /tmp/lima/laborbenko/etc
	$(MAKE) create name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko"

.ssh :
	cp -r ~/.ssh .ssh

wireguard :
	cp -r ../wireguard wireguard

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

.PHONY : clean
clean :
	rm -rf wireguard
	rm -rf .ssh
	rm -rf /tmp/lima/laborbenko/etc

.PHONY : clean-al
clean-all : clean
	rm -rf images/*
