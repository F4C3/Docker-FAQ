# Docker-FAQ

---

## Dockerfile

```
FROM ubuntu:latest - образ машины 
RUN apt-get update - выполнить консольную команду 
RUN apt-get install -y nginx - выполнить консольную команду 
WORKDIR /data - рабочая директория в контейнере
VOLUME /data - директория на хосте кторорая пробрасывается в контейнер
EXPOSE 80 - открывает порт
CMD ["nginx", "-g", "daemon off;"] - CMD — инструкция для запуска чего-либо во время запуска самого контейнера. 
```
## Консоль

`docker build . -t test-nginx ` - Построить образ из Dockerfile

`docker images` - Посмотреть список образов

`doсker stop` - остановить любой котнейнер

`docker rm` - удалить любой контейнер

`docker rmi bd57e967757` - Удалить любой из образов (bd57e967757 - IMAGE ID) 

`docker run -d -p 80:80 -v C:\Users\a.pryakhin\code:/data/mysite.local/:rw test-nginx` - Эта команда в фоновом режиме (-d) запустит контейнер на базе образа test-nginx и пробросом портов (-p) 80->80, а также общей директорией (-v) из C:\Users\a.pryakhin\code на хосте в /data/mysite.local в контейненере с правами на запись и чтение (:rw).

`docker images` - просмото имеющихся видов

`docker ps` - список работающих контейнеров, их ID, имена и т.д

`docker exec -it [CONTAINER ID] /bin/bash` - подключится к контейнеру по id ( b2edc8b47a63 )

#### Сеть
```
docker network ls -  список сетей

docker network inspect  [CONTAINER ID] | [CONTAINER NAME] - Проверка контейнера, получение как можно большего количества информации о контейнере, от портов, переменных среды до точек монтирования, данных контрольной группы и т. д.

docker network create [имя сети] - создать сеть в Docker
```
`docker run --rm -it --name container1 --net myNetwork nicolaka/netshoot /bin/bash` - Пример команды где --rm удаляет контейнер после закрытия, --name задаёт имя контейнера (не путать с CONTAINER ID ), --net помещает контейнер в соответствующую сеть, nicolaka/netshoot репозиторий docker с сетевыми утилитами, /bin/bash команда входа на созданный контейнер


---

