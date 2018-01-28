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
