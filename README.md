# Astra
##Ansible

**Первоначальная настройка**

```bash
apt update
apt install -y ansible

mkdir -p /etc/ansible
ansible-config init --disabled > /etc/ansible/ansible.cfg

**/etc/ansible/ansible.cfg**

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

#Подготовка директорий и пользователя
useradd --system --create-home --home-dir /var/lib/semaphore --shell /usr/sbin/nologin semaphore
mkdir -p /etc/semaphore /var/lib/semaphore/playbooks
chown -R semaphore:semaphore /etc/semaphore /var/lib/semaphore

#Первоначальная настройка (semaphore setup)
sudo -u semaphore semaphore setup --config /etc/semaphore/config.json

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

#Создание службы systemd для автозапуска

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

systemctl daemon-reload
systemctl enable --now semaphore
systemctl status semaphore

#Файл настроек
nano /etc/semaphore/config.json

#Смена пароля администратора при необходимости
sudo -u semaphore semaphore user add \
  --config /etc/semaphore/config.json \
  --admin \
  --login admin \
  --name "Administrator" \
  --email "admin@itadmin.su" \
  --password 'ВашНовыйПароль123!'

```



