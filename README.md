# Учебный стенд Elasticsearch с буфером Apache Kafka и Fluentd в качестве источника данных

## Описание:

Данный [плейбук](playbook.yml) выполняет конфигурацию учебного стенда. Описание станда в файле [inventory/hosts.ini]().

Стенд состоит из 7-ми хостов. На 3 ноды группы **es** устанавливается [elasticsearch](roles/elasticsearch/tasks/main.yml), на **log-aggregator** запускаются [apache kafka](roles/kafka/tasks/main.yml), [td-agent](roles/fluentd/tasks/main.yml) и [kibana](roles/kibana/tasks/main.yml). На серверах группы **app** устанавливается только td-agent.

## Data stream

На серверах прложений fluentd вычитывает логи веб-сервера nginx (_/var/log/nginx_) и логи контейнеров docker (_/var/lib/docker/containers_). Полученные логи отправляются в брокер kafka на сервере log-aggregator. log-aggregator забирает логи из kafka и отправляет их в кластер elasticsearch.

Подробнее в файлах настроек td-agent: [apps](roles/fluentd/templates/td-agent-app.conf.j2), [log-aggregator](roles/fluentd/templates/td-agent-logs.conf.j2).

## Применение

Перед запуском заполнить адреса в файле [inventory/hosts.ini]() (ansible_hosts).

Запуск:
```
~/elastic-kafka-fluentd$ ansible-playbook playbook.yml
```

После первого запуска заполнить получить значения внутренних IP адресов, прервать выполнение, и заполнить **vars** в [плейбуке](playbook.yml):
- es1_ip: 10.110.0.8
- es2_ip: 10.110.0.3
- es3_ip: 10.110.0.5
- kafka_ip: 10.110.0.2

После этого запустить повторно.

Ожидание в 10 минут необходимо, чтобы корректно отработал обход блокировок apt/dpkg.

## Заметки

Установка паролей системным учёткам кластера:

Первичный запуск elastic

```
~# /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive/auto
```

Сброс пароля пользователя кластера elastic

```
~# /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic/kibana_system
```

Просмотр сообщений в топике kafka

```
~# /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka_IP:9092 --topic topic1 topic2 --from-beginning
```

Удаление топика

```
~# /opt/kafka/bin/kafka-topics.sh --delete --bootstrap-server kafka_IP:9092 --topic your_topic_name
```
