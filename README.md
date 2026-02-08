# ansible-vps

Запуск контейнера с терминалом:
docker compose run --rm ansible

## Запуск апгрейда:
ansible-playbook -i hosts playbook-debian-12-to-13.yml

## Установка openConnect:
ansible-playbook -i hosts playbook-openconnect.yml

Для проверки используйте --check


## Создать пользователя (старый вариант)
ocpasswd -c /etc/ocserv/ocpasswd username.

### После настройки Радиус и БД проверить пользователей и создать

mysql -u radius -p radius -e "SELECT * FROM radcheck;"

INSERT INTO radcheck (username, attribute, op, value) VALUES ('testuser', 'Cleartext-Password', ':=', 'testpass');