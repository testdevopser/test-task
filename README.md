# test-task (Тестовое задание)

## Запуск
*Предполагается что вы запускаете это на Linux Ubuntu (я делал всё на версии 22.04) и у Вас есть уже установленны браузер, docker, ansible.

Для запуска на локальной машине нужно: 
1) скачать этот репозиторий
```
git clone https://github.com/testdevopser/test-task.git
```

2) Зайти в папку test-task и запустить плейбук
````
ansible-playbook playbook.yml -K
````
3) Запустить в папке test-task (в которой находится файл docker-compose.yml) 

```
docker compose up -d
```


3) Открыть в браузере
   http://localhost - тут будет Grafana (логин и пароль дефолтные admin/admin)
   и там должен быть импортирован дашборд ![Filebeat Overview (beat-exporter)](image-1.png)





## Описание

playbook.yml - устанавливает и настраивает 2 вещи:  
1) Filebeat для аггрегации и записи логов в "/var/log/test" с указанием имени docker контейнера в логе
пример вывода /var/log/test
```
[grafana] logger=context userId=0 orgId=0 uname= t=2025-06-22T10:42:56.537390521Z level=info msg="Request Completed" method=GET path=/ status=302 remote_addr=172.18.0.1 time_ms=0 duration=671.429µs size=29 referer=http://localhost/ handler=/ status_source=server 
[grafana] logger=authn.service t=2025-06-22T10:42:56.541456204Z level=warn msg="Failed to authenticate request" client=auth.client.session error="user token not found" 
[grafana] logger=provisioning.dashboard type=file name=default t=2025-06-22T10:43:00.120735254Z level=error msg="failed to load dashboard from " file=/etc/grafana/provisioning/dashboards_files/counter.json error="invalid character '\"' after object key:value pair" 
[grafana] logger=authn.service t=2025-06-22T10:43:01.77521334Z level=warn msg="Failed to authenticate request" client=auth.client.session error="user token not found"
```
2) и также оно устанавливает такую штуку как [beat-exporter] (https://github.com/trustpilot/beat-exporter.git) - это надо чтобы дать метрики prometheus в понятном ему виде. (напрмямую у меня не получилось скормить prometheus вывод filebeat). 

Также допилил ещё и systemd сервис на его основе (/etc/systemd/system/beat-exporter.service)
````
[Unit]
Description=Trustpilot Beat Exporter for Prometheus
After=network.target

[Service]
ExecStart=/usr/local/bin/beat-exporter -beat.uri "http://localhost:5066" -web.listen-address "0.0.0.0:9479"
Restart=always
User=root

[Install]
WantedBy=multi-user.target

````
Он запускает бинарник beat-exporter это небольшой прокси, который опрашивает Filebeat по его встроенному HTTP API и экспортирует эти данные в формате Prometheus metrics. Так что этот beat-exporter работает как прокладка между Filebeat API и самим Prometheus.

## Описание дашборда 
![Filebeat Overview (beat-exporter](image-2.png)

   Тут использованы label_values для заполнения переменных. Панель мониторинга состоит из трех строк: краткие ключевые показатели эффективности, пропускная способность событий и статус ошибок/очереди, чтобы обеспечить четкое представление всех основных показателей.


### Ключевые особенности этого дашборда:
 - Обзор KPI: Верхняя панель показывает самые важные показатели: статус, время работы, количество активных горутин и общее количество обработанных событий.
 - Производительность конвейера: Графики показывают скорость обработки событий (опубликованные, подтвержденные, отфильтрованные), что помогает выявить "узкие места".
- Ошибки и очередь: Отдельные панели для отслеживания ошибок отправки и количества событий в очереди на отправку (показатель задержек).
- Потребление ресурсов: Графики использования CPU и памяти.
Активность Harvester и Registrar: Показывает, сколько файлов читается в данный момент и как часто обновляется состояние.
Секция для модуля Auditd: Сворачиваемая секция для метрик, специфичных для модуля auditd.
 
## Структура

```
├── docker-compose.yml
├── grafana
│   └── provisioning
│       ├── dashboards
│       │   └── dashboards.yml
│       ├── dashboards_files
│       │   └── dashboard.json
│       └── datasources
│           └── prometheus.yml
├── playbook.yml
├── prometheus
│   └── prometheus.yml
├── README.md
└── roles
    ├── beat_exporter
    │   ├── defaults
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── beat-exporter.service.j2
    └── filebeat_docker_logs
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── filebeat.yml.j2

```
