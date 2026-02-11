# ansible-vps

Запуск контейнера с терминалом:
docker compose run --rm ansible

## Запуск апгрейда:
ansible-playbook -i hosts playbook-debian-12-to-13.yml

## Установка openConnect:
ansible-playbook -i hosts playbook-openconnect.yml

## Установка openConnect:
ansible-playbook -i hosts playbook-radius.yml

Для проверки используйте --check


## Создать пользователя (старый вариант)
ocpasswd -c /etc/ocserv/ocpasswd username.

### После настройки Радиус и БД проверить пользователей и создать

mysql -u radius -p radius -e "SELECT * FROM radcheck;"

INSERT INTO radcheck (username, attribute, op, value) VALUES ('testuser', 'Cleartext-Password', ':=', 'testpass');

### Если включили pap md5

INSERT INTO radcheck (username, attribute, op, value) VALUES ('testuser', 'MD5-Password', ':=', 'testpass');

UPDATE radcheck 
SET attribute = 'MD5-Password', value = MD5('testpass') 
WHERE username = 'testuser';