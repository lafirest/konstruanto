.PHONY : build
build : .ssh wireguard .build
qiname := laboro
tag := init
layer := init
ip := "10.0.0.2"
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
create : docker-env
	@echo lima nerdctl run -it --privileged \
		--network=laboro \
		--mount type=volume,source=laboro,destination=/mnt/laboro \
		$(mount) \
		--name $(name) laborbenko:$(tag) -l $(layer) -i $(ip) -e $(eval) -b $(isBuild)

.PHONY : docker-env
docker-env : docker-network docker-volume

.PHONY : %-image
%-image : images/%-image
	@

.PHONY : %-image-test
%-image-test : images/%-image-test
	@

#.PRECIOUS: images/%-image
images/%-image : 	ip := $(shell wireguard/bin/alloc_ip true)
images/%-image : etc/%
	@echo "building the image: $*"
	mkdir -p /tmp/lima/laborbenko
	rm -rf /tmp/lima/laborbenko/etc
	cp -r etc /tmp/lima/laborbenko/etc
	$(MAKE) create name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko" layer=$* isBuild=true ip=$(ip)
	@echo lima nerdctl commit laborbenko-$*-image laborbenko:$*
	@echo lima nerdctl container rm laborbenko-$*-image
	@touch $@

images/%-image-test :
	@echo "building the image: $*"
	mkdir -p /tmp/lima/laborbenko
	rm -rf /tmp/lima/laborbenko/etc
	cp -r etc /tmp/lima/laborbenko/etc
	$(MAKE) create name=laborbenko-$*-image mode=-it mount="-v /tmp/lima/laborbenko/etc:/etc/laborbenko"

images/init-image: .ssh wireguard Dockerfile entrypoint
	lima nerdctl build -t laborbenko:init .
	@touch $@

images/emqx-image : tag=erlang
images/emqx-image : images/erlang-image

images/erlang-image : tag=base
images/erlang-image : images/base-image

images/base-image : images/init-image

.ssh :
	cp -r ~/.ssh .ssh

wireguard :
	cp -r ../wireguard wireguard

.build : Dockerfile entrypoint etc/*
	lima nerdctl build -t laborbenko:init .
	touch .build
RESULT := $(shell lima nerdctl network ls --format '{{.Name}}' | grep laboro)
.PHONY : docker-network
docker-network :
ifeq ($(RESULT),)
	lima nerdctl network create laboro
endif

.PHONY : test
test :
	lima nerdctl run -it --privileged --network=laboro --rm --name laborbenko laborbenko:init
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
