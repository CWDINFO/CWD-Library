---
description: Сборка это такой процесс, когда из исходных кодов получается программа.
---

# Сборка ядра crowdwiz-core

Перед сборкой нужно поставить необходимые библиотеки и настроить операционную систему. Все дальнейшим команды мы будем выполнять для виртуальной машины, которую мы создали в облаке Яндекса, однако для дроплета DO эти команды также подойдут.

## Общая настройка системы

Подключимся к серверу, который мы создали в предыдущей части:

`ssh witness@ваш_ip_адрес`

Для начала нужно обновить систему и поставить все необходимые библиотеки, это делается с помощью нескольких команд:

`sudo apt update`

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

а затем: `sudo apt upgrade`

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Здесь нужно ввести Y и нажать Enter.

После этого запустится процесс обновления системы, который может длиться несколько минут.

Теперь нужно создать SWAP-файл, он позволит нам сэкономить на хостинге. Дело в том, что в данный момент узлу, производящему блоки для работы 8 GB оперативной памяти более чем достаточно. Однако чтобы осуществить сборку ядра этой памяти недостаточно. Чтобы не арендовать сервер на 16 GB, которые будут нужны только при сборке, мы используем технологию файла подкачки (SWAP), которая позволит расширить оперативную память в момент сборки проекта за счёт свободного места на диске сервера.

Сначала отключим файл подкачки если он уже есть: `sudo swapoff /swapfile`

Затем создадим новый файл подкачки на 8 GB, каждая команда вводится отдельно:\
`sudo fallocate -l 8G /swapfile`

`sudo chmod 600 /swapfile`

`sudo mkswap /swapfile`

`sudo swapon /swapfile`

`sudo swapon --show`

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Если вы делали это на виртуальной машине от Яндекса, то на этом всё, если на другом хостинге (например, на DO), то может потребоваться ввести ещё одну команду:\
`echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab`\
Она нужна чтобы после перезагрузки наш файл подкачки создавался автоматически. Теперь нужно поставить необходимые для сборки библиотеки и программы:\
`sudo apt-get install autoconf cmake make automake libtool`

`git libboost-all-dev libssl-dev g++ libcurl4-openssl-dev screen`

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

дальше будет большой список библиотек, который должен заканчиваться вопросом

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

Нужно ввести Y и нажать Enter. После того как установка будет завершена, можно переходить непосредственно к сборке витнеса.

## Сборка crowdwiz-core

Сначала мы должны скачать исходные коды с гитхаба, затем сконфигурировать сборку систему сборки и непосредственно собрать. Поскольку процесс сборки занимает продолжительное время (в первый раз это может происходить до 2-х часов), то для того чтобы избежать прерываний сборки в случае обрывов связи, мы воспользуемся утилитой screen. Она создаст ещё один терминал, который будет работать независимо от того, подключены мы в данный момент к серверу или нет. И даже если соединение будет разорвано, мы сможем снова подключиться к запущенному сеансу.

Выполняем команду:\
`screen`\
И видим окно сообщение от создателей программы:

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

затем нажимаем Enter.

Теперь мы в скрине, который визуально ничем не отличается от терминала, в котором мы работали до сих пор. Теперь нам нужно получить исходные файлы проекта с github. Для этого нужно выполнить команду:\
`git clone https://github.com/crowdwiz-biz/crowdwiz-core.git`

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

у нас появился каталог с исходными файлами, теперь нужно в него войти:\
`cd crowdwiz-core`\
проверить что мы находимся на мастер ветке:\
`git checkout master`\
и запустить скрипт конфигурации:\
`cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo`

Теперь можно запускать сборку, для этого нужно написать команду:\
`make`\
Вы увидите много бегущих строк, и это будет продолжаться довольно долго - на сервере Яндекса примерно 1 час. Если за это время вы отключались от сервера, то нужно будет вновь подключиться и вернутся в консоль, в которой происходила сборка. Это можно сделать с помощью команд:\
`ssh witness@ваш_ip_адрес screen -r`

После того как сборка завершена, можно приступать к настройке самой ноды.
