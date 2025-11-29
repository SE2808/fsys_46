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

### Веб-сайт через балансировщик
- **URL**: http://158.160.211.70
- **Проверка**: `curl http://158.160.211.70`

### Zabbix (Мониторинг)
- **URL**: http://62.84.127.225
- **Проверка**: `curl -I http://62.84.127.225`

### Kibana (Визуализация логов)
- **URL**: http://158.160.97.174:5601
- **Проверка**: `curl -I http://158.160.97.174:5601`

### Bastion host
- **IP**: 158.160.108.103

## Развертывание инфраструктуры

### 1. Установка и настройка Terraform
```bash
wget https://hashicorp-releases.yandexcloud.net/terraform/1.7.3/terraform_1.7.3_linux_amd64.zip
zcat terraform_1.7.3_linux_amd64.zip > terraform
chmod 766 terraform
./terraform -v
vim ~/.terraformrc
chmod 664 .terraformrc
```

### 2. Настройка Yandex Cloud CLI
```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
yc init
yc config list
yc iam create-token
```

### 3. Генерация SSH ключей
```bash
ssh-keygen
cat ~/.ssh/id_ed25519.pub
```

### 4. Инициализация и применение Terraform
```bash
./terraform init
./terraform apply
```

## Настройка Ansible

### Установка и конфигурация
```bash
sudo apt install -y ansible
mkdir -p ~/ansible
cd ansible/
# Настройка конфигурационных файлов:
# - ansible.cfg
# - hosts.ini
```

### Добавление SSH ключей на все ВМ
```bash
KEY_CONTENT=$(cat ~/.ssh/id_ed25519.pub)
for vm in vm-web-1 vm-web-2 vm-bastion vm-zabbix vm-elastic vm-kibana; do
  echo "Добавляем ключ в $vm"
  yc compute instance add-metadata $vm --metadata ssh-keys="$KEY_CONTENT"
done
```

### Проверка подключения
```bash
ansible bastion -m ping
ansible all -m ping
ansible-inventory --list -i hosts.ini
```

## Развертывание сервисов через Ansible

### Настройка nginx на веб-серверах
```bash
ansible-playbook -i hosts nginx-setup.yaml
# Проверка: curl http://158.160.211.70
```

### Установка Zabbix сервера
```bash
ansible-playbook -i hosts zabbix-playbook.yaml
# Проверка службы:
ansible -i hosts zabbix -m systemd -a "name=zabbix-server state=started" -b
```

### Настройка Zabbix агентов
```bash
ansible-playbook -i hosts zabbix-agent-setup.yml
```

### Установка Elasticsearch
```bash
ansible-playbook -i hosts elastic-play.yml
# Проверка службы:
ansible -i hosts elasticsearch -m systemd -a "name=elasticsearch state=started" -b
```

### Установка Kibana
```bash
ansible-playbook -i hosts kibana-play.yaml
# Проверка службы:
ansible -i hosts kibana -m systemd -a "name=kibana state=started" -b
```

### Настройка Filebeat для сбора логов
```bash
ansible-playbook -i hosts filebeat-play.yaml
# Проверка службы:
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

## Доступ для проверки
Предоставлен доступ ко всем веб-интерфейсам:

- **[Сайт через балансировщик](http://158.160.211.70)**
- **[Zabbix с настроенными дашбордами](http://62.84.127.225)**
- **[Kibana с логами nginx](http://158.160.97.174:5601)**

---