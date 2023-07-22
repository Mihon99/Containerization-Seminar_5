## Урок 5. Docker Compose и Docker Swarm

### **Информация о проекте**
Задание:
1) создать сервис, состоящий из 2 различных контейнеров: 1 - веб, 2 - БД
2) далее необходимо создать 3 сервиса в каждом окружении (dev, prod, lab)
3) по итогу на каждой ноде должно быть по 2 работающих контейнера
4) выводы зафиксировать

**Выполнение**
# Создаем файл docker-composer.yaml
root@reaper:~/compose# nano compose.yaml

# Посмотрим содержимое файла
root@reaper:~/compose# cat compose.yaml
version: ‘3.9’

services:

  db:
    image: mariadb:10.10.2
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 12345

  adminer:
    image: adminer:4.8.1
    restart: always
    ports:
      - 6080:8080

# Поднимаем сервис на основе этого файла
mss@mssvm:~/dz5$ sudo docker compose up -d
[+] Running 20/20
 ✔ adminer 7 layers [⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                             55.6s 
   ✔ 34df401c391c Pull complete                                                                                                                                                  52.0s 
   ✔ 942860e9b081 Pull complete                                                                                                                                                  53.7s 
   ✔ f571177b537e Pull complete                                                                                                                                                  53.7s 
   ✔ 78d7a59571f8 Pull complete                                                                                                                                                  53.8s 
   ✔ 530e7e02f755 Pull complete                                                                                                                                                  53.8s 
   ✔ 03ee8734c62c Pull complete                                                                                                                                                  53.9s 
   ✔ ed7a0cc37cf2 Pull complete                                                                                                                                                  53.9s 
 ✔ db 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                                                            111.5s 
   ✔ e2c03c89dcad Pull complete                                                                                                                                                  61.2s 
   ✔ 68eb43837bf8 Pull complete                                                                                                                                                  61.2s 
   ✔ 796892ddf5ac Pull complete                                                                                                                                                  61.3s 
   ✔ 6bca45eb31e1 Pull complete                                                                                                                                                  61.5s 
   ✔ ebb53bc0dcca Pull complete                                                                                                                                                  61.5s 
   ✔ 2e2c6bdc7a40 Pull complete                                                                                                                                                  61.6s 
   ✔ 6f27b5c76970 Pull complete                                                                                                                                                  92.4s 
   ✔ 438533a24810 Pull complete                                                                                                                                                  92.4s 
   ✔ e5bdf19985e0 Pull complete                                                                                                                                                 108.5s 
   ✔ 667fa148337b Pull complete                                                                                                                                                 108.5s 
   ✔ 5baa702110e4 Pull complete                                                                                                                                                 108.5s 
[+] Running 3/3
 ✔ Network dz5_default      Created                                                                                                                                               0.1s 
 ✔ Container dz5-adminer-1  Started                                                                                                                                               1.0s 
 ✔ Container dz5-db-1       Started                                                                                                                                               1.0s 

# Проверяем 
mss@mssvm:~/dz5$ sudo docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
a2c763b17f62   mysql:8.0       "docker-entrypoint.s…"   53 seconds ago   Up 52 seconds   3306/tcp, 33060/tcp                         dz5-db-1
68ae79a13683   adminer:4.8.1   "entrypoint.sh php -…"   53 seconds ago   Up 52 seconds   0.0.0.0:8085->8080/tcp, :::8085->8080/tcp   dz5-adminer-1

# Подключаемся к adminer  в браузере по адресу http://localhost:8085

# Инициализируем swarm
mss@mssvm:~/dz5$ sudo docker swarm init
Swarm initialized: current node (jb5fd4hyolcmmkgjwj1en6w56) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4qyahk52c4qgjjre56iekn4p8kg94ovfksz6ectd1h0nyapvjl-a6314lnvqhx6ojqtmh53hsjpk 192.168.1.48:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

mss@mssvm:~/dz5$ 

# Добавляем вторую ноду на хосте mssvm2
mss@mssvm2:~$ docker swarm join --token SWMTKN-1-4qyahk52c4qgjjre56iekn4p8kg94ovfksz6ectd1h0nyapvjl-a6314lnvqhx6ojqtmh53hsjpk 192.168.1.48:2377
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/swarm/join": dial unix /var/run/docker.sock: connect: permission denied
mss@mssvm2:~$ sudo !!
sudo docker swarm join --token SWMTKN-1-4qyahk52c4qgjjre56iekn4p8kg94ovfksz6ectd1h0nyapvjl-a6314lnvqhx6ojqtmh53hsjpk 192.168.1.48:2377
[sudo] password for mss: 
This node joined a swarm as a worker.

# Добавляем третью ноду на хосте mssvm3
mss@mssvm3:~$ sudo docker swarm join --token SWMTKN-1-4qyahk52c4qgjjre56iekn4p8kg94ovfksz6ectd1h0nyapvjl-a6314lnvqhx6ojqtmh53hsjpk 192.168.1.48:2377
[sudo] password for mss: 
This node joined a swarm as a worker.

# Проверяем наличие нод и их состояние (на хосте мастер-ноды)
mss@mssvm:~/dz5$ sudo docker node ls
ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
jb5fd4hyolcmmkgjwj1en6w56 *   mssvm      Ready     Active         Leader           24.0.4
latu4gl026bleo789vc6vpyk1     mssvm2     Ready     Active                          24.0.4
pn1c1mwrfleovi5nulkrsclox     mssvm3     Ready     Active                          24.0.4

# Добавляем нодам метки(labels) и проверяем их наличие
mss@mssvm:~/dz5$ sudo docker node update --label-add env=lab pn1c1mwrfleovi5nulkrsclox
pn1c1mwrfleovi5nulkrsclox
mss@mssvm:~/dz5$ sudo docker inspect --format='{{json .Spec}}' pn1c1mwrfleovi5nulkrsclox
{"Labels":{"env":"lab"},"Role":"worker","Availability":"active"}

mss@mssvm:~/dz5$ sudo docker node update --label-add env=dev jb5fd4hyolcmmkgjwj1en6w56
jb5fd4hyolcmmkgjwj1en6w56
mss@mssvm:~/dz5$ sudo docker inspect --format='{{json .Spec}}' jb5fd4hyolcmmkgjwj1en6w56
{"Labels":{"env":"dev"},"Role":"manager","Availability":"active"}

mss@mssvm:~/dz5$ sudo docker node update --label-add env=prod latu4gl026bleo789vc6vpyk1
latu4gl026bleo789vc6vpyk1
mss@mssvm:~/dz5$ sudo docker inspect --format='{{json .Spec}}' latu4gl026bleo789vc6vpyk1
{"Labels":{"env":"prod"},"Role":"worker","Availability":"active"}

# Создаем сервис sql-service на двух контейнерах
mss@mssvm:~/dz5$ docker service create --name sql-service --replicas 2 -e MYSQL_ROOT_PASSWORD=tst123 mysql:8.0
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/services/create": dial unix /var/run/docker.sock: connect: permission denied
mss@mssvm:~/dz5$ sudo !!
sudo docker service create --name sql-service --replicas 2 -e MYSQL_ROOT_PASSWORD=tst123 mysql:8.0
jfi651hjbbdhi0ugdj1hhz0rz
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged 

# Проверяем наличие и распределение контейнеров сервиса на всех нодах
mss@mssvm:~/dz5$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
5b1f6b31eadc   mysql:8.0       "docker-entrypoint.s…"   2 minutes ago    Up 2 minutes    3306/tcp, 33060/tcp                         sql-service.2.s5fkd4yl4608da2dbfcizf60x
a2c763b17f62   mysql:8.0       "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   3306/tcp, 33060/tcp                         dz5-db-1
68ae79a13683   adminer:4.8.1   "entrypoint.sh php -…"   19 minutes ago   Up 19 minutes   0.0.0.0:8085->8080/tcp, :::8085->8080/tcp   dz5-adminer-1
mss@mssvm2:~$ sudo docker ps
[sudo] password for mss: 
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS          PORTS                                       NAMES
8ac1fe7a8206   mysql:8.0       "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes    3306/tcp, 33060/tcp                         sql-service.1.cipnv7s3865hnk735mv3aisup
e5e722524b4c   mysql:8.0       "docker-entrypoint.s…"   2 hours ago     Up 25 minutes   3306/tcp, 33060/tcp                         dz5-db-1
b435bed9c721   adminer:4.8.1   "entrypoint.sh php -…"   2 hours ago     Up 25 minutes   0.0.0.0:8085->8080/tcp, :::8085->8080/tcp   dz5-adminer-1
mss@mssvm3:~$ sudo docker ps
[sudo] password for mss: 
CONTAINER ID   IMAGE           COMMAND                  CREATED       STATUS          PORTS                                       NAMES
e5e722524b4c   mysql:8.0       "docker-entrypoint.s…"   2 hours ago   Up 28 minutes   3306/tcp, 33060/tcp                         dz5-db-1
b435bed9c721   adminer:4.8.1   "entrypoint.sh php -…"   2 hours ago   Up 28 minutes   0.0.0.0:8085->8080/tcp, :::8085->8080/tcp   dz5-adminer-1

# Масштабируем сервис в 0 (снимаем нагрузку)
mss@mssvm:~/dz5$ sudo docker service scale sql-service=0

# Создаем новый сервис из 2-х контейнеров на ноде "prod"
mss@mssvm:~/dz5$ sudo docker service create --name prod_sql_service --constraint node.labels.env==prod --replicas 2 -e MYSQL_ROOT_PASSWORD=tst123 mysql:8.0
[sudo] password for mss: 
1u8wniwtkdcngtjt5k6slfniw
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged

# Проверяем сервисы
mss@mssvm:~/dz5$ sudo docker service ls
ID             NAME               MODE         REPLICAS   IMAGE       PORTS
1u8wniwtkdcn   prod_sql_service   replicated   2/2        mysql:8.0   
jfi651hjbbdh   sql-service        replicated   0/0        mysql:8.0 

# Проверяем контейнеры на ноде "prod" (хост mssvm2)
mss@mssvm2:~/dz5$ sudo docker ps
[sudo] password for mss: 
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                 NAMES
6025739096d3   mysql:8.0   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp, 33060/tcp   prod_sql_service.1.nqjpr1mjcjjv36wkbcy9cfu7k
124c74d8a0b5   mysql:8.0   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp, 33060/tcp   prod_sql_service.2.meih3kv60brskmtvjvtg05uxj
