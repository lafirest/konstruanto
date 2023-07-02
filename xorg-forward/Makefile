build : Dockerfile
	docker build -t xorg-forward:0.1.0 .

.PHONY : run
run :
	docker run --name=xorg-forward --hostname=xorg-forward -d --network=laboro -p 127.0.0.1:3389:3389 xorg-forward:0.1.0


.PHONY : stop
stop :
	docker container stop xorg-forward
	docker container rm xorg-forward
