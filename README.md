# test-task (Тестовое задание)

playbook.yml - устанавливает и настраивает 2 вещи:  Filebeat для аггрегации и записи логов в "/var/log/test" и также оно устанавливает такую штуку как beat-exporter (https://github.com/trustpilot/beat-exporter.git) - это надо чтобы дать метрики prometheus в понятном ему виде. (напрмямую у меня не получилось скормить prometheus вывод filebeat)

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
