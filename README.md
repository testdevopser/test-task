# test-task (Тестовое задание)

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
3) Открыть в браузере
   http://localhost - тут будет Grafana (логин и пароль дефолтные admin/admin)
   и там должен быть импортирован дашборд ![Filebeat Overview (beat-exporter)](image-1.png)

   Тут использованы label_values для заполнения переменных. Панель мониторинга состоит из трех строк: краткие ключевые показатели эффективности, пропускная способность событий и статус ошибок/очереди, чтобы обеспечить четкое представление всех основных показателей.

## Ключевые особенности этого дашборда:
 - Обзор KPI: Верхняя панель показывает самые важные показатели: статус, время работы, количество активных горутин и общее количество обработанных событий.
 - Производительность конвейера: Графики показывают скорость обработки событий (опубликованные, подтвержденные, отфильтрованные), что помогает выявить "узкие места".
- Ошибки и очередь: Отдельные панели для отслеживания ошибок отправки и количества событий в очереди на отправку (показатель задержек).
- Потребление ресурсов: Графики использования CPU и памяти.
Активность Harvester и Registrar: Показывает, сколько файлов читается в данный момент и как часто обновляется состояние.
Секция для модуля Auditd: Сворачиваемая секция для метрик, специфичных для модуля auditd.
 
## Описание
playbook.yml - устанавливает и настраивает 2 вещи:  
1) Filebeat для аггрегации и записи логов в "/var/log/test" с указанием имени docker контейнера в логе
пример вывода /var/log/test
```
[grafana] logger=context userId=0 orgId=0 uname= t=2025-06-22T10:42:56.537390521Z level=info msg="Request Completed" method=GET path=/ status=302 remote_addr=172.18.0.1 time_ms=0 duration=671.429µs size=29 referer=http://localhost/ handler=/ status_source=server 
[grafana] logger=authn.service t=2025-06-22T10:42:56.541456204Z level=warn msg="Failed to authenticate request" client=auth.client.session error="user token not found" 
[grafana] logger=provisioning.dashboard type=file name=default t=2025-06-22T10:43:00.120735254Z level=error msg="failed to load dashboard from " file=/etc/grafana/provisioning/dashboards_files/counter.json error="invalid character '\"' after object key:value pair" 
[grafana] logger=authn.service t=2025-06-22T10:43:01.77521334Z level=warn msg="Failed to authenticate request" client=auth.client.session error="user token not found" 
[grafana] logger=context userId=1 orgId=1 uname=admin t=2025-06-22T10:43:02.018663281Z level=info msg="Request Completed" method=GET path=/api/live/ws status=-1 remote_addr=172.18.0.1 time_ms=2 duration=2.89734ms size=0 referer= handler=/api/live/ws status_source=server 
[grafana] logger=provisioning.dashboard type=file name=default t=2025-06-22T10:43:10.135948505Z level=error msg="failed to load dashboard from " file=/etc/grafana/provisioning/dashboards_files/counter.json error="invalid character '\"' after object key:value pair" 
[grafana] logger=context userId=1 orgId=1 uname=admin t=2025-06-22T10:43:12.546646422Z level=info msg="Request Completed" method=GET path=/api/live/ws status=-1 remote_addr=172.18.0.1 time_ms=2 duration=2.963233ms size=0 referer= handler=/api/live/ws status_source=server 
[grafana] logger=provisioning.dashboard type=file name=default t=2025-06-22T10:43:20.127572853Z level=error msg="failed to load dashboard from " file=/etc/grafana/provisioning/dashboards_files/counter.json error="invalid character '\"' after object key:value pair" 
[grafana] logger=provisioning.dashboard type=file name=default t=2025-06-22T10:43:30.134220134Z level=error msg="failed to load dashboard from " file=/etc/grafana/provisioning/dashboards_files/counter.json error="invalid character '\"' after object key:value pair" 
```
2) и также оно устанавливает такую штуку как  ![beat-exporter] (https://github.com/trustpilot/beat-exporter.git) - это надо чтобы дать метрики prometheus в понятном ему виде. (напрмямую у меня не получилось скормить prometheus вывод filebeat). Поэтому допилил

```
.
├── docker-compose.yml
├── grafana
│   └── provisioning
│       ├── dashboards
│       │   └── dashboards.yml
│       ├── dashboards_files
│       │   ├── my_dashboard.json
│       │   └── sample.json
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
