# Домашнее задание к занятию "ELK" - `Яковлев Артем`

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---

### Дополнительные ресурсы

При выполнении задания используйте дополнительные ресурсы:
- [docker-compose elasticsearch + kibana](11-03/docker-compose.yaml);
- [поднимаем elk в docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html);
- [поднимаем elk в docker с filebeat и docker-логами](https://www.sarulabs.com/post/5/2019-08-12/sending-docker-logs-to-elasticsearch-and-kibana-with-filebeat.html);
- [конфигурируем logstash](https://www.elastic.co/guide/en/logstash/7.17/configuration.html);
- [плагины filter для logstash](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html);
- [конфигурируем filebeat](https://www.elastic.co/guide/en/beats/libbeat/5.3/config-file-format.html);
- [привязываем индексы из elastic в kibana](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html);
- [как просматривать логи в kibana](https://www.elastic.co/guide/en/kibana/current/discover.html);
- [решение ошибки increase vm.max_map_count elasticsearch](https://stackoverflow.com/questions/42889241/how-to-increase-vm-max-map-count).

## Задание 1. Elasticsearch 

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный. 

*Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name*.

### Установка (локальная) и запуск Elasticsearch
```
sudo apt update && sudo apt install gnupg apt-transport-https
#Доступ к ресурсам elastic.co из РФ заблокирован.
#Пакет скачал через web browser (proxy-addon) по адресу: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-amd64.deb
sudo apt install /tmp/elasticsearch-7.6.2-amd64.dep
sudo systemctl daemon-reload
sudo systemctl status elasticsearch.service
sudo systemctl enable elasticsearch.service
sudo sysctl vm.swappiness=1 #или выключаем подкачку: sudo swapoff -a

sudo nano /etc/elasticsearch/elasticsearch.yml # cluster.name: clusterBaranovskiiSN и network.host: localhost
sudo systemctl start elasticsearch.service
curl -X GET 'localhost:9200/_cluster/health?pretty'
curl -X GET 'localhost:9200/_cat/master?pretty'
curl -X GET 'http://localhost:9200'
```
![Скриншот команды](https://github.com/temagraf/ELK/blob/main/img/11-3-1.png "Скриншот команды")

---

## Задание 2. Kibana

Установите и запустите Kibana.

*Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty*.

### Установка и запуск в docker-е Kibana и Elasticsearch
```
sudo apt install -y docker docker-doc docker-compose
sudo usermod -aG docker $USER
sudo chmod -R 666 /var/run/docker.sock
docker info
docker-compose -f docker-compose.yaml up -d
docker ps

#
# Или ставим локально (скачан https://artifacts.elastic.co/downloads/kibana/kibana-7.17.9-amd64.deb)
#
sudo apt install /tmp/kibana-7.17.9-amd64.deb
sudo systemctl daemon-reload
sudo systemctl status logstash.service
sudo systemctl enable logstash.service
sudo systemctl start logstash.service

sudo nano /etc/kibana/kibana.yml # server.host: "localhost"  и  server.port: 5601

http://localhost:5601/app/dev_tools#/console
```
![Скриншот интерфейса Kibana](https://github.com/temagraf/ELK/blob/main/img/11-3-2.png "Скриншот интерфейса Kibana")

---

## Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.*

```
sudo apt install nginx
sudo systemctl status nginx.service

#Доступ к ресурсам elastic.co из РФ заблокирован. В docker в том числе.
#Пакет скачал через web browser (proxy-addon) по адресу: https://artifacts.elastic.co/downloads/logstash/logstash-7.17.9-amd64.deb
sudo apt install /tmp/logstash-7.17.9-amd64.deb
sudo systemctl daemon-reload
sudo systemctl status logstash.service
sudo systemctl enable logstash.service
sudo systemctl start logstash.service

sudo chmod -R o+r /var/log/nginx # Для доступа к логам на чтение
sudo nano /etc/logstash/conf.d/my_pipelines.conf
```
![Скриншот интерфейса Kibana](https://github.com/temagraf/ELK/blob/main/img/11-3-3.png "Скриншот интерфейса Kibana")

---

## Задание 4. Filebeat. 

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.*

```
#Доступ к ресурсам elastic.co из РФ заблокирован.
#Пакет скачал через web browser (proxy-addon) по адресу: https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.9-amd64.deb
sudo apt install /tmp/filebeat-7.17.9-amd64.deb
sudo systemctl daemon-reload
sudo systemctl status filebeat.service
sudo systemctl enable filebeat.service

sudo nano /etc/filebeat/filebeat.yml
sudo filebeat modules enable system nginx

sudo systemctl start filebeat.service
```

---

## Задание 5*. Доставка данных 

Настройте поставку лога в Elasticsearch через Logstash и Filebeat любого другого сервиса , но не Nginx. 
Для этого лог должен писаться на файловую систему, Logstash должен корректно его распарсить и разложить на поля. 

*Приведите скриншот интерфейса Kibana, на котором будет виден этот лог и напишите лог какого приложения отправляется.*

### Настройка поставки лога Logstash ( /var/log/logstash/logstash-deprecation.log )

```
sudo chmod -R o+r /var/log/logstash # Для доступа к логам на чтение
sudo filebeat modules enable logstash # Добавляем модуль logstash

sudo nano /etc/logstash/conf.d/my_pipelines.conf 
sudo nano /etc/filebeat/filebeat.yml 

sudo systemctl restart logstash.service
sudo systemctl resrart filebeat.service

```
- **[My_pipelines_filebeat.conf](https://github.com/temagraf/ELK/tree/main/11-03/my_pipelines_filebeat.conf)**
- **[Filebeat.yml](https://github.com/temagraf/ELK/tree/main/11-03/filebeat.yml)**


---
