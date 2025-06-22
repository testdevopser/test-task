# test-task (Тестовое задание)

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
