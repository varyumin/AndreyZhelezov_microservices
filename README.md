#######################################################

# docker-6. Домашнее задание #19.

## Подготовка Gitlab CI.
На ресурсах GCP была создана виртуальная машина с помощью docker-machine с использованием драйвера google. На ВМ было дополнительно установлено docker-compose, а так же созданы диретктории необходимые для работы gitlab и установлен gitlab-ce в контейнере. После чего интеррфейс gitlab стал доступен по внешнему адресу виртуальной машины из интернет.
После смены пароля root, успешно авторизовавшись, отключили регистрацию новых пользователей.
## Создание проекта.
Через интерфейл Gitlab была создана группа **homework** и проект **example**. После чего проект был подключен как удаленный ресурс к git-репозиторию в ветке docker-6.

#######################################################

# docker-4. Домашнее задание #17.
## Работа с сетью Docker.
### Сетевой драйвер "none".
Для проверки работы сетевого драйвера "none" использован образ с предустановленными сетевыми утилитами **joffotron/docker-net-tools**. На его основе создан и запущен контейнер. В процессе его работы, подключившись к контейнеру в интерактивном режиме видим что внутри контейнера есть loopback интефейс.
### Сетевой драйвер "host". 
Для проверки работы сетевого драйвера "host" контейнер запущен с параметром _--network host_. После чего были сравнены выводы двух команд _ifconfig_ для запущеных с созданного контейнера "net_test" и затем с хоста "docker-host". Т.к. драйвер "host" использует сетевой стек хоста внутри контейнеров, выводы этих двух команд почти идентичны, вывод _ifconfig_ изнутри контейнера только дополнительно указывает к какому порту прибинден текущий сетевой стек контейнера:
```
...
docker0   Link encap:Ethernet  HWaddr 02:42:DB:90:4C:57       |	docker0   Link encap:Ethernet  HWaddr 02:42:db:90:4c:57  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:25	          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:25
          inet6 addr: fe80::42:dbff:fe90:4c57%*32691*/64 Scope: |	          inet6 addr: fe80::42:dbff:fe90:4c57/64 Scope:Link
...
```  
Так же для проверки работы драйвера "host" были несколько раз созданы контейнеры nginx: _docker run --network host -d nginx_. В результате каждый следующий контейнер создаваясь пытался занять всё тот же порт tcp/80 хоста что и предыдущий, в результате процесс nginx контейнера завершался с ошибкой, а с ним и сам контейнер переставал работать.
### Сетевой драйвер "bridge". 
#### Работа внутри одной сети.
Для проверки работы серевого драйвера "bridge" были запущены контейнеры с микросервисами нашего приложения, как и в прошлом ДЗ, но без указания сетевых алиасов, поэтому сервисы не могли общаться друг с другом. После указания алиасов при перезапуске контейнеров приложение заработало.
#### Работа в двух сетях.
Были созданы две сети типа "bridge": _back_net_ для контейнера MongoDB  и _front_net_ для ui. Таким образом у фронтэнда не будет доступа к БД. Промежуточные сервисы приложения (их контейнеры) были добавлены изначально только в сеть _back_net_, но при таком варианте у сервисов post и comment небыло доступа к БД. Поэтому уже после создания и запуска контейнеров они (контейнеры post и comment сервисов) были добавлены и во вторую сеть _front_net_. После чего приложение успешно заработало.
## Docker-compose.
На рабочем компьютере установлена утилита **docker-compose**. В каталоге reddit-microservices репозиория с ДЗ создан файл docker-compose.yml. В него добавлено описание всей инфраструктуры контейнеров приложения в т.ч. volume, сеть и сами контейнеры. Далее с использованием этого файла была создана инфраструктура, предварительно была определена переменная окружения $USERNAME с именем пользователя в docker-hub. Вход на веб-интерфейс приложения показал что оно работает. Однако при попытке перейти в созданный пост получаем ошибку приложения. И это не удивительно в docker-compose.yml, у сервиса post_db как минимум не хватает сетевого алиаса comment_db, поэтому сервис comment не знает как обратиться к базе данных. Исправил это недоразумение тем что добавил алиасы в раздел _networks_ сервиса _post_db_. 
### Самостоятельное задание.
1. Описание проекта в файле docker-compose.yml изменено т.о. что сервисы разделены по двум сетям  _back_net_ и _front_net_ и сервису mongo_db назначены необходимые сетевые алиасы. Тестирование показало что приложение полностью работоспособно.
2. Параметризированы некоторые значения в файле docker-compose.yml:
* USERNAME - имя пользователя,
* H_PORT_UI - порт хоста на который сбинден порт сервиса UI,
* C_PORT_UI - порт сервиса UI внутри контейнера,
* UI_VER - версия образа для сборки контейнера ui,
* POST_VER - версия образа для сборки контейнера post,
* COMMENT_VER - версия образа для сборки контейнера comment.
3. Переменные окружения применяемые в docker-compose.yml описаны в файле .env проекта. Образец такого файла выложен в репозиторий под именем .env.example
4. Переменные применяются из этого файла автоматически при условии запуска docker-compose из директории в которой он лежит.

#######################################################

# docker-3. Домашнее задание #16.
Все рабочие файлы текущего репозитория перенесены в каталог **./docker-monolith/**. В корень репозитория добавлен файл **.dockerignore** с исключениями из контекста в т.ч. указанной директории.
## Разбиение приложения на микросервисы.
В корень репозитория распакован каталог с приложением: **reddit-microservices**.
В каталоги с именами частей приложения: **post**, **comment**, **ui** помещены Dockerfile-ы с описанием образов. На основе этих Docerfile были контейнеры для запуска приложения, каждый сервис в своём контейнере. Так же был скачан контейнер с последней версией MongoDB. P.S. сборка контейнера ui началась с первого шага.
Далее была создана одноранговая сеть _reddit_ средствами docker. В неё будут добавлены наши контейнеры с приложением. 
Все контейнеры были запущены в работу в детач режиме. Каждому контейнеру кроме ui присвоены network-алиасы. Они исполняют роль доменных имен, по которым к контейнерам можно обращаться.
Для проверки я подключился к внешнему адресу docker-host на экспонированый порт 9292 к веб-интерфейсу приложения и создал один пост.
## Оптимизация контейнеров.
В процессе пересборки образов неоднократно возникала ошибка нехватки памяти на docker-host. Поэтому тип машины docker-host на GCP был изменен на g1-small.
Dockerfile сервиса ui был изменен и на его основе был собран образ на >300Mb меньше образа первой версии:
```
azhelezov/ui            2.0                 75b9f41ab998        32 seconds ago      453MB
azhelezov/ui            1.0                 630b8791ede8        2 days ago          775MB
```
Данным образом был создан новый контейнер ui и приложение запущено с этим новым контейнером. После перезапуска контейнера с MongoDB все данные записанные ранее в БД пропали, и поста созданого на предыдущем шаге небыло. Чтобы это исправить был создан иподключен volume для хранения данных. Чтобы смонтировать данный volume в директорию БД в контейнере mongo, все контейнеры были перезапущены, а в команду запуска контейнера mongo была добавлена привязка созданного volume к этому контейнеру опция _-v reddit_db:/data/db_.
Для проверки работы volume на запущеном приложении был создан пост в веб-интерфейсе и далее контейнеры были снова перезапущены. На этот раз после перезапуска информация в БД сохранилась - пост был на месте.

#######################################################

# docker-2. Домашнее задание #15.
## Исходные данные.
ОС Ubuntu Linux (xenial) 16.04.
Установлена docker-machine версии 0.13.0.
В GCP создан новый проект: docker-XXXXXXXX. К нему перенастроена конфигурация gcloud.
## Работа с docker-host.
### Создание docker-host.
На GCP с помощью gcloud создана и запущена ВМ с названием docker-host. Проверка командой _docker-machine ls_ показала что хост создался и что он запущен.
Изменено окружение docker-machine на демона docker-host. Таким образом все команды docker будут выполняться демоном на docker-host.
### вопрос со \*.
Разница в выводе двух комманд:
* docker run --rm -ti tehbilly/htop;
* docker run --rm --pid host -ti tehbilly/htop;

,состоит в том что в первом (дефолтном) варинате мы увидим процесс htop созданного контейнера и только его, а во втором случае мы увидим все процессы хоста с которого запускаем контейнер. Происходит так потому что по дефолту контейнер запускается со своим **PID Namespace** и единственный процесс в нем получает PID=1, а во втором случае из-за опции _--pid=host_ процессу из контейнера выделяется PID из общего namespace хоста. И вцелом из контейнера мы получаем доступ к **PID namespace** хоста и видим все его процессы.
### Создание образа с приложением.
В репозитории создан файл конфигурации MongoDB: mongod.conf.
Создан bash-скрипт запуска приложения: start.sh.
Создан файл переменных в котором задается переменная IP-адреса хоста с БД: db_conf. 
Создан файл Dockerfile, в который будет размещено опсание нашего образа. В Dockerfile добавлены следующие комманды:
* задание исходного образа
```
FROM ubuntu:16.04
```
* обновление кэша репозиториев и установка пакетов для работы приложения
```
RUN apt-get update
RUN apt-get install -y mongodb-server ruby-full ruby-dev build-essential git
RUN gem install bundler
```
* закачка приложения в контейнер
```
RUN git clone https://github.com/Artemmkin/reddit.git
```
* копирование файлов конфигурации в контейнер
```
COPY mongod.conf /etc/mongod.conf
COPY db_config /reddit/db_config
COPY start.sh /start.sh
```
* установка зависимостей и насройка прав доступа к скрипту запуска приложения
```
RUN cd /reddit && bundle install
RUN chmod 0777 /start.sh
```
* выполнение старта приложения при старте контейнера
```
CMD ["/start.sh"]
```
На docker-host cоздан образ контейнера **reddit:latest** из описания в Dockerfile. На основе образа **reddit:latest** создан и запущен контейнер. 
При попытке подключиться к приложению, запущеному в контейнере, по внешнему адресу хоста на порт 9292 соединение не устанавливалось из-за отсутствия соответствующего правила фаервола в VPC-разделе GCP. После добавления правила доступ к приложению по http/9292 появился.
## Работа с Docker Hub.
Создана учетная запись на Docker Hub. С использованием этой учетной записи осуществлен логин из контекста docker к Docker Hub.
Далее созданный ранее образ **reddit:latest** был залит на Docker Hub для дальнейшего использования. Для этого локальному образу образу был присвоен тэг удаленного образа **<login_name>/otus-reddit:1.0** и сделан push к удаленному образу.

#######################################################

# docker-1. Домашнее задание #14.
## Исходные данные
ОС Ubuntu Linux (xenial) 16.04.
Установлена community версия Docker 17.12. Тестовый запуск показал что клиентская и серверная часть работают правильно.
## Операции с контейнерами.
После запуска тестового контейнера "Hello world" испробованы команды Docker для вывода списка контейнеров, образов. Затем на основе образа Ubuntu:16.04 было создано два контейнера и опробованы различные опции команды *docker run*.
Далее были опробованы команды для подключения к уже запущеному контейнеру и комбинации выхода из контейнера без его закрытия.
После чего был создан образ на основе контейнера, модифицированного созданием файла /tmp/file. Вывод команды docker images в котором виден созданный образ сохранен в файл docker-1.log.
### задание со *
Сравнение вывода двух команд docker inspect <image_ID> и docker inspect <container_ID> дописано в файл docker-1.log.

Далее испробованы команды завершения работы контейнеров, отображения занимаемого дискового пространства, удаления контейнеров и образов с опциями.

# AndreyZhelezov_microservices
