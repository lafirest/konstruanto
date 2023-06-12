.PHONY : build
build : .ssh wireguard .build

.ssh :
	cp -r ~/.ssh .ssh

wireguard :
	cp -r ../wireguard wireguard

.build : Dockerfile entrypoint etc/*
	lima nerdctl build -t laborbenko:init .
	touch .build

.PHONY : test
test :
	lima nerdctl run -it --privileged --network=laboro --rm --name laborbenko laborbenko:init

.PHONY : clean
clean :
	rm -rf wireguard
	rm -rf .ssh
