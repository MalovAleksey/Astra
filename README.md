# Astra
## Ansible

**Первоначальная настройка**

```bash
apt update
apt install -y ansible sshpass
```
```bash
mkdir -p /etc/ansible
ansible-config init --disabled > /etc/ansible/ansible.cfg
```
**/etc/ansible/ansible.cfg**
```bash
[defaults]
inventory = /etc/ansible/hosts
host_key_checking = False
interpreter_python = auto_silent
remote_user = admin

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True

```
**Установка semaphore**
```bash
wget https://github.com/semaphoreui/semaphore/releases/download/v2.20.23/semaphore_2.20.23_linux_amd64.deb
apt install ./semaphore_2.20.23_linux_amd64.deb
```

```bash
#Подготовка директорий и пользователя
useradd --system --create-home --home-dir /var/lib/semaphore --shell /usr/sbin/nologin semaphore
mkdir -p /etc/semaphore /var/lib/semaphore/playbooks
chown -R semaphore:semaphore /etc/semaphore /var/lib/semaphore
```
```bash
#Первоначальная настройка (semaphore setup)
sudo -u semaphore semaphore setup --config /etc/semaphore/config.json
```
```bash
#What database to use:
4

#db Hostname (default /tmp/database.sqlite):
/var/lib/semaphore/database.sqlite

#Playbook path (default /tmp/semaphore):
/var/lib/semaphore/playbooks

#Public URL (optional, example: https://example.com/semaphore): Чтобы открывался но IP:3000 оставьте пустым.
https://semaphore.itadmin.su

Enable email alerts? (yes/no) (default no):
Enable telegram alerts? (yes/no) (default no):
Enable slack alerts? (yes/no) (default no):
Enable Rocket.Chat alerts? (yes/no) (default no):
Enable Microsoft Team Channel alerts? (yes/no) (default no):
Enable LDAP authentication? (yes/no) (default no):
```
#Создание службы systemd для автозапуска

```bash
tee /etc/systemd/system/semaphore.service > /dev/null <<EOF
[Unit]
Description=Semaphore UI (Ansible Web UI)
Documentation=https://semaphoreui.com  
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=semaphore
Group=semaphore
ExecStart=/usr/bin/semaphore server --config /etc/semaphore/config.json
Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
EOF
```
```bash
systemctl daemon-reload
systemctl enable --now semaphore
systemctl status semaphore
```
#Файл настроек
nano /etc/semaphore/config.json

```bash
#Смена пароля администратора при необходимости
sudo -u semaphore semaphore user add \
  --config /etc/semaphore/config.json \
  --admin \
  --login admin \
  --name "Administrator" \
  --email "admin@itadmin.su" \
  --password 'ВашНовыйПароль123!'

```

## Обновление Astra Linux Desktop с 1.7 до 1.8

Подготовка пакетов

```bash
apt install astra-full-upgrade fly-notifications fly-update-notifier-major -y
apt policy astra-full-upgrade
astra-full-upgrade self-upgrade
apt policy astra-full-upgrade
```

Редактируем файл, при необходимости меняем репозитории.

```bash
nano /usr/lib/python3/dist-packages/astra_upgrade/configs/upgrade.conf.yaml

```
или 
```bash
nano /etc/astra-upgrade/configs/upgrade.conf.yaml
```
```bash
nesting_upgrade: true
```
```bash
check_gpg: false
```
Выполняем проверку

```bash
astra-full-upgrade check
```

Запускаем обновление

```bash
astra-full-upgrade enable
```
или

```bash
astra-full-upgrade force
```


## Локальные репозитории

```bash
 rsync --delete -avLz --info=progress2 dl.astralinux.ru::astra/astra/stable/2.12_x86-64/repository/ /srv/repo/alce
```















