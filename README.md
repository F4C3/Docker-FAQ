# Docker-FAQ

---

## Dockerfile

```
FROM ubuntu:latest - образ операционной системы 
RUN apt-get update - выполнить консольную команду 
RUN apt-get install -y nginx - выполнить консольную команду 
WORKDIR /data - рабочая директория в контейнере
VOLUME /data - директория на хосте кторорая пробрасывается в контейнер
EXPOSE 80 - открывает порт
CMD ["nginx", "-g", "daemon off;"] - CMD — инструкция для запуска чего-либо во время запуска самого контейнера. 
```
## Консоль

`docker build . -t test-nginx ` - Построить образ из Dockerfile

`docker rmi bd57e967757` - Удалить любой из образов (bd57e967757 - IMAGE ID) 

`docker images` - Посмотреть список образов

`doсker stop` - остановить любой котнейнер

`docker rm` - удалить любой контейнер

`docker run -d -p 80:80 -v C:\Users\a.pryakhin\code:/data/mysite.local/:rw test-nginx` - Эта команда в фоновом режиме (-d) запустит контейнер на базе образа test-nginx и пробросом портов (-p) 80->80, а также общей директорией (-v) из C:\Users\a.pryakhin\code на хосте в /data/mysite.local в контейненере с правами на запись и чтение (:rw).

`docker images` - просмотр имеющихся видов

`docker ps` - список работающих контейнеров, их ID, имена и т.д

`docker exec -it [CONTAINER ID] /bin/bash` - подключится к контейнеру по id ( b2edc8b47a63 )

#### Сеть
```
docker network ls -  список сетей

docker network inspect  [CONTAINER ID] | [CONTAINER NAME] - Проверка контейнера, получение как можно большего количества информации о контейнере, от портов, переменных среды до точек монтирования, данных контрольной группы и т. д.

docker network create [имя сети] - создать сеть в Docker

docker network connect [имя или id сети] [имя или id контейнера]- подключить контейнер к ещё одной сети

Отключится от сети

1. docker inspect [имя или id контейнера] - Узнать NetworkID или имя сети которую нужно отключить

2. network disconnect [имя или id сети][имя или id контейнера] - отключить контейнер от сети
```
`docker run --rm -it --name container1 --net myNetwork nicolaka/netshoot /bin/bash` - Пример команды где --rm удаляет контейнер после выхода из него, --name задаёт имя контейнера (не путать с CONTAINER ID ), --net помещает контейнер в соответствующую сеть, nicolaka/netshoot репозиторий docker с сетевыми утилитами, /bin/bash это Shell оболочка для запуска, -it это сокращение от --interactive + --tty. Когда вы запускаете докер с помощью этой команды, вы попадаете прямо внутрь контейнера.

***Docker Compose — это инструмент для управления многоконтейнерными Docker-средами. В нём используется YML-файл для настройки создания и взаимодействия контейнеров.***

### docker-compose.yaml

```
# версия синтаксиса
version: '3'
 
# в этом блоке мы описываем контейнеры, которые будут запускаться
services:
 
  #Контейнер с PHP, назовём его app
  app:
    # Если нет секции build, то система будет искать образ в репозиториях
    build:
      context: ./fpm
      dockerfile: Dockerfile
    image: myapp/php # имя будущего образа
    container_name: app # имя контейнера после запуска
    volumes:
       - ./code:/data/mysite.local
    # мы можем создать для контейнеров внутреннюю сеть
    networks:
      - app-network
 
  #контейнер с Nginx
  webserver:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    image: myapp/nginx
    container_name: webserver
    volumes:
       - ./code:/data/mysite.local

    # проброс портов
    ports:
      - "80:8080"
      - "443:443"
    networks:
      - app-network
 
  # контейнер с MySQL
  # строим на базе стандартного образа
  db:
    image: mysql:5.7.22
    container_name: db
    ports:
      - "3306:3306"
    # описываем, какую БД мы создаём
    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE} # переменная из .env
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD} # переменная из .env
    volumes:
      - ./dbdata:/var/lib/mysql  
    networks:
      - app-network
 
#Docker Networks
networks:
  app-network:
    driver: bridge
```

### .env

***.env это отдельный файл, в котором хранятся секретные данные. Такой файл никогда не коммитится в основной репозиторий, а значения из него попадают в docker-compose только в момент сборки контейнеров. В примере указаны переменные с данными о логине и пароле базы данных***

```
MYSQL_DATABASE=laravel
MYSQL_ROOT_PASSWORD=laravel
```

`docker-compose up -d` - запуск контейнера из docker-compose.yml

***Обратите внимание на то, что при возникновении ошибок или при изменении конфигурации Dockerfile одного из контейнеров может возникнуть потребность в пересборке (помните, что Docker всегда старается по максимуму брать данные из кэша). Тогда команда должна выглядеть вот так:*** `docker-compose up -d -build`


---

