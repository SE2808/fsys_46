# <p align="center">Дипломный проект: Отказоустойчивая инфраструктура в Yandex Cloud</p>

**FSYS-46 / Ларионов Сергей**

---

## Описание проекта
Развертывание отказоустойчивой инфраструктуры для веб-сайта с мониторингом, сбором логов и резервным копированием в Yandex Cloud.

## Архитектура
- **Веб-серверы**: 2 VM в разных зонах (nginx)
- **Балансировщик**: Yandex Application Load Balancer
- **Мониторинг**: Zabbix сервер + агенты на всех хостах
- **Сбор логов**: Elasticsearch + Kibana + Filebeat
- **Сеть**: VPC с NAT-шлюзом, Security Groups
- **Доступ**: Bastion host для SSH подключений

## Доступ к сервисам

### Веб-сайт
```bash
curl http://158.160.211.70
```
**Балансировщик**: 158.160.211.70

### Zabbix (Мониторинг)
**URL**: http://62.84.127.225/zabbix

### Kibana (Визуализация логов)
**URL**: http://158.160.97.174:5601

### Bastion host
**IP**: 158.160.108.103

## Развертывание инфраструктуры

### 1. Инициализация Terraform
```bash
./terraform init
```

### 2. Применение конфигурации
```bash
./terraform apply
```

## Ansible плейбуки

### Настройка nginx
```bash
ansible-playbook -i hosts nginx-setup.yaml
```

### Установка Zabbix сервера
```bash
ansible-playbook -i hosts zabbix-playbook.yaml
```

### Настройка Zabbix агентов
```bash
ansible-playbook -i hosts zabbix-agent-setup.yml
```

### Установка Elasticsearch
```bash
ansible-playbook -i hosts elastic-play.yml
```

### Установка Kibana
```bash
ansible-playbook -i hosts kibana-play.yaml
```

### Настройка Filebeat
```bash
ansible-playbook -i hosts filebeat-play.yaml
```

## Проверка служб

### Проверка Zabbix
```bash
ansible -i hosts zabbix -m systemd -a "name=zabbix-server state=started" -b
curl -I http://62.84.127.225/zabbix
```

### Проверка Elasticsearch
```bash
ansible -i hosts elasticsearch -m systemd -a "name=elasticsearch state=started" -b
```

### Проверка Kibana
```bash
ansible -i hosts kibana -m systemd -a "name=kibana state=started" -b
curl -I http://158.160.97.174:5601
```

### Проверка Filebeat
```bash
ansible -i hosts nginx-web -m systemd -a "name=filebeat state=started" -b
```

## Сетевые настройки

### Internal IP адреса:
- **web-1**: 192.168.10.10
- **web-2**: 192.168.10.20  
- **elasticsearch**: 192.168.10.30
- **bastion**: 192.168.40.10
- **zabbix**: 192.168.40.20
- **kibana**: 192.168.40.30

### External IP адреса:
- **bastion**: 158.160.108.103
- **zabbix**: 62.84.127.225
- **kibana**: 158.160.97.174
- **load balancer**: 158.160.211.70

## Особенности реализации
- Использование прерываемых ВМ для экономии (перед проверкой переведены в постоянные)
- Все ВМ без внешних IP, кроме bastion, zabbix, kibana и балансировщика
- Настройка Security Groups для ограничения доступа
- Ежедневное резервное копирование снапшотов с TTL 7 дней
- Мониторинг по метрикам USE (Utilization, Saturation, Errors)

## Доступ для проверки
Предоставлен доступ ко всем веб-интерфейсам:
- Сайт через балансировщик
- Zabbix с настроенными дашбордами
- Kibana с логами nginx

---