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

`docker exec -it [CONTAINER ID] /bin/bash` - подключится к контейнеру по id 

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

---
<details>
 <summary>Если надо получить доступ к контейнеру по ip c хоста Windows</summary>
</br>
 
 Если мы с хоста попробуем пропинговать IP контейнера в сети front, то есть выполнить в консоли ping 172.19.0.3, то мы нифига не увидим. Контейнер оказывается не доступен. Как сделать контейнер доступным по IP с хоста? Это на самом деле не тривиальная задача.

 Чтобы связать внутреннюю сеть докера и хост мы должны создать новый статический маршрут. Для начала посмотрим список уже созданных маршрутов. В этом нам поможет команда route. Выполняем на виндовом хосте:
 
```
 $ route print
 [...]
 ===========================================================================
 Постоянные маршруты:
   Сетевой адрес            Маска    Адрес шлюза      Метрика
         172.0.0.0        255.0.0.0        10.0.75.2       1
 ===========================================================================
 [...]
 ```
 Вывод довольно здоровый, я оставил только значимую часть, там еще сверху и снизу от вырезанного куска по экрану текста. У вас скорее всего блок «Постоянные маршруты» будет пустой. У меня он уже создан, и поэтому команда route print его показывает.

 Если у вас такого нет, то добавляем новый маршрут. Следующую команду нужно выполнять из под админа. То есть нужно запустить консоль с правами администратора и только потом выполнять
```
 $ route /P add 172.0.0.0 MASK 255.0.0.0 10.0.75.2
  ОК
```
 
 Статический маршрут во внутреннюю сеть докера создан. Но это еще не все. Во всех новых версиях докера виртуалка с линуксом запрещает доступ по IP контейнера. Ранее это было возможно, но на текущий момент по дефолту запрещено правилами iptables. Официального метода не существует, разработчики докера считают что это никому не нужно / очень сложно реализовать. Для понимания проблемы почитайте вот [это](https://docs.docker.com/engine/userguide/networking/default_network/container-communication/#container-communication-between-hosts) в официальной документации, раздел «Container communication between hosts».

 Для фикса этого поведения нужно выполнить команду:

 `$ docker run --rm -it --privileged --network=none --pid=host justincormack/nsenter1 bin/sh -c "iptables -P FORWARD ACCEPT"`
 Вот. Теперь есть и статический маршрут до внутренней сети докера и правила на виртуалке разрешают такие подключения. Эта команда если вкратце изменяет правила iptables на виртуалке докера. Пробуем пропинговать контейнер по его IP из сети front, команда ping 172.19.0.3 теперь будет показывать корректный обмен пакетами.

 `docker run --rm -it --name container1 --net myNetwork nicolaka/netshoot /bin/bash` - Пример команды где --rm удаляет контейнер после выхода из него, --name задаёт имя контейнера (не путать с CONTAINER ID ), --net помещает контейнер в соответствующую сеть, nicolaka/netshoot репозиторий docker с сетевыми утилитами, /bin/bash это Shell оболочка для запуска, -it это сокращение от --interactive + --tty. Когда вы запускаете докер с помощью этой команды, вы попадаете прямо внутрь контейнера.
 
Дополнительные пояснения по маршруту
 
Еще раз пройдемся по всем пунктам. Мы создали сеть front с драйвером bridge. Это все штатные команды докера, тут ничего особенного нет, все должно быть понятно. Подробнее разберем всю дальнейшую магию.

Контейнеры на винде стартуют на виртуалке. То есть докер создает виртуальную машину, и вообще все контейнеры запускаются именно на виртуалке. У виртуалки есть статический IP адрес, как правило этот адрес 10.0.75.2, вы можете безо всяких маршрутов его попробовать пропинговать. Этот адрес спокойно пингуется. Все сети которые создает докер создаются внутри виртуалки. IP контейнера nginx в сети front 172.19.0.3, это значит, что на докеровской виртуалке создана сеть в которой для контейнера nginx выделен определенный ip адрес. Во внешем мире об этом IP адресе никто не знает.

Когда мы создаем статический маршрут командой

`$ route /P add 172.0.0.0 MASK 255.0.0.0 10.0.75.2`
 ОК
Мы говорим винде, что маршрут до всех IP адресов начинающихся с цифры 172 надо строить через IP адрес 10.0.75.2. Если пользователю вдруг нужен ипшник 172.19.0.3, то винда будет пытаться достучаться до него через 10.0.75.2, или другими словами будет искать этот адрес внутри виртуалки.

И последняя магическая деталь касательно сети. Команда

`$ docker run --rm -it --privileged --network=none --pid=host justincormack/nsenter1 bin/sh -c "iptables -P FORWARD ACCEPT"`
для изменения правил iptables. Докеровская виртуалка по дефолту всех посылает нахрен, говорит что нужного IP адреса у нее нет. И вот эта команда меняет дефолтное поведение виртуалки. После ее выполнения виртуалка будет корректно достраивать оставшуюся часть маршрута. Офигенный минус в том, что это временное решение. При переустановке докера или еще во многих других случаях, когда машина создается заново, докер забывает эту команду. То есть ее периодически придется выполнять заново. Статический маршрут на хостовой винде вы один раз забили и все, можно про это забыть. А вот это вот поведение забыть не получится. Если вдруг перестали проходить пинги до контейнеров, скорее всего докер забыл про эту команду и надо ее выполнить заново. Если после ее выполнения пинги так и не пошли, значит дело в чем то другом.
 
 </details>
 
 ---
 
***Docker Compose — это инструмент для управления многоконтейнерными Docker-средами. В нём используется YML-файл для настройки создания и взаимодействия контейнеров.***

## docker-compose.yml

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

***Для остановки многоконтейнерного приложения в той же директории, что и docker-compose, вызовите*** `docker-compose down`

### Commit

***Зафиксировать изменения внесённые в контейнер***

```
$ docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS              NAMES
c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            desperate_dubinsky
197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours                            focused_hamilton

$ docker commit c3f279d17e0a  svendowideit/testimage:version3

f5283438590d

$ docker images

REPOSITORY                        TAG                 ID                  CREATED             SIZE
svendowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB
```

---

