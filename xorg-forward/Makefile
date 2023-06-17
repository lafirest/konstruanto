build : Dockerfile
	lima nerdctl build -t xorg-forward:0.1.0 .

.PHONY : run
run :
	lima nerdctl run --name=xorg-forward --hostname=xorg-forward -d --network=laboro -p 127.0.0.1:3389:3389 xorg-forward:0.1.0


.PHONY : stop
stop :
	lima nerdctl container stop xorg-forward
	lima nerdctl container rm xorg-forward
