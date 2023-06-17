name := laboro
tag := init
layer := init
eval := ""
isBuild := false

.PHONY : run
run :
	lima nerdctl run -it --privileged \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--rm --name $(name) laborbenko:$(tag) -l $(layer) -e $(eval) -b $(isBuild)

.PHONY : create
create :
	lima nerdctl run -it --privileged \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--name $(name) laborbenko:$(tag) -l $(layer) -e $(eval) -b $(isBuild)

.PHONY : docker-env
docker-env : docker-network docker-volume

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
	$(MAKE) create name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true tag=$(tag)
	lima nerdctl commit laborbenko-$*-image laborbenko:$*
	lima nerdctl container rm laborbenko-$*-image
	@touch images/$*-image

.ssh :
	cp -r ~/.ssh .ssh

wireguard :
	cp -r ../wireguard wireguard

bin : entrypoint wireguard
	mkdir -p bin
	cp entrypoint bin/
	cp wireguard/bin/* bin/

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
	rm -rf bin

.PHONY : clean-al
clean-all : clean
	rm -rf images/*
