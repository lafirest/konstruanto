build : Dockerfile
	cp -r ../wireguard wireguard
	cp -r ~/.ssh .ssh
	lima nerdctl build -t laborbenko:init .

.PHONY : test
test :
	lima nerdctl run -it --privileged --network=laboro --rm --name laborbenko laborbenko:init

.PHONY : clean
clean :
	rm -rf wireguard
	rm -rf .ssh
