-docker run -it --device-write-bps /dev/sda:1mb ubuntu /bin/bash
=>/dev/sda 디바이스에 대해 1mb만 쓸 수 있도록 설정한다.

-dd if=/dev/urandom of=test.out bs=1mb count=10 oflag=direct
=>용량에 대한 제한을 걸 수 있다.


docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest

     1  version: "3.9"
     2
     3  services:
     4    db:
     5      image: mysql:5.7
     6      volumes:
     7        - db_data:/var/lib/mysql
     8      restart: always
     9      environment:
    10        MYSQL_ROOT_PASSWORD: somewordpress
    11        MYSQL_DATABASE: wordpress
    12        MYSQL_USER: wordpress
    13        MYSQL_PASSWORD: wordpress
    14
    15    wordpress:
    16      depends_on:
    17        - db    =>db가 동작해야지만 wordpress가 동작함. 종료될 땐 workpress가 종료된 후 db가 종료.
    18      image: wordpress:latest
    19      ports:
    20        - "8000:80"
    21      restart: always
    22      environment:
    23        WORDPRESS_DB_HOST: db:3306
    24        WORDPRESS_DB_USER: wordpress
    25        WORDPRESS_DB_PASSWORD: wordpress
    26        WORDPRESS_DB_NAME: wordpress
    27  volumes:
    28    db_data: {}





    docker-compose   --version
docker  --version

docker-compose  file  구조
version  <--  https://docs.docker.com/compose/compose-file/compose-versioning
services   <--  컨테이너 설정(docker  container   run  ..)
networks  <--  도커 네트워크 설정(docker  network  create  ..)
volumes  <--  도커 영구볼륨(docker  volume  create  ...)



cd   ~/docker_lab/compose/wordpress_compose
cat -n docker-compose.yaml


root@docker1:~/docker_lab/compose/wordpress_compose# docker ps -a
CONTAINER ID   IMAGE              COMMAND                  CREATED             STATUS             PORTS                                   NAMES
dbdd2a2a6fc3   centos             "/bin/ping localhost"    35 seconds ago      Up 35 seconds                                              testping
c6bff4395014   wordpress:latest   "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:8000->80/tcp, :::8000->80/tcp   wordpress_compose_wordpress_1
df198d22c174   mysql:5.7          "docker-entrypoint.s…"   About an hour ago   Up About an hour   3306/tcp, 33060/tcp                     wordpress_compose_db_1
a7a6125b8c78   ubuntu             "bash"                   3 hours ago         Up 3 hours 



root@docker1:~/docker_lab/compose/wordpress_compose# docker-compose ps -a
            Name                           Command               State                  Ports
-------------------------------------------------------------------------------------------------------------
wordpress_compose_db_1          docker-entrypoint.sh mysqld      Up      3306/tcp, 33060/tcp
wordpress_compose_wordpress_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:8000->80/tcp,:::8000->80/tcp

=>docker-compose명령어는 yaml파일로 만들어진 것만 보여줌.

##Docker Swarm에서의 Service
=>같은 이미지로 생성된 컨테이너 집합이며, 제어 가능한 최소 단위.

manager노드에서 워크노드로 "이런것좀 해줘" 하는부분이 tasks?


root@docker1:~# docker service ps firstweb
ID             NAME         IMAGE                 NODE      DESIRED STATE   CURRENT STATE           ERROR     PORTS
mjt3qhb7pfet   firstweb.1   takytaky/app:latest   node1     Running         Running 3 minutes ago

=>docker swarm에서 만들어진것만 보여줌.


docker   service   create   --name  sweb  --replicas 2  -p  80:80   nginx
	->  docker   service   ls
	->  docker   service  ps  sweb

윈도우 웹브라우저 -  http://

docker   service   scale  sweb=6
	->  docker   service  ps  swarm

=>도커스웜에서 global로 하면 하나의 노드에 하나의 컨테이너만 뛰움. 중앙집중화 모니터링할 때 사용함.


root@docker1:~# docker   service   create  --mode  global  --name  logsrv   nginx^C
root@docker1:~# docker service ps logsrv
ID             NAME                               IMAGE          NODE           DESIRED STATE   CURRENT STATE            ERROR     PORTS
zqu66z98e8wp   logsrv.b55c5u0zeydts0xr81ri38qa6   nginx:latest   node2          Running         Running 45 seconds ago
w0exlsfbf1i4   logsrv.g3xwpvqhqtwbk0syv1zzq3ws2   nginx:latest   manager-node   Running         Running 45 seconds ago
ciluu34g4q2h   logsrv.nhgw2vd7onhcfk7qok2aryfia   nginx:latest   node1          Running         Running 45 seconds ago

=>하나씩 생성됨. global모드


root@node2:~# docker swarm join-token manager
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
=>예전 정보가 남아있어서 join이 되지 않음. 이전 정보 삭제 후 다시 join 수행.

