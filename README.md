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

### Если включили pap md5 - не рекомендуется легко ломается по rainbow tables

INSERT INTO radcheck (username, attribute, op, value) VALUES ('testuser', 'MD5-Password', ':=', 'testpass');

UPDATE radcheck 
SET attribute = 'MD5-Password', value = MD5('testpass') 
WHERE username = 'testuser';

### SHA2 любой голый хэш тоже уязвим для rainbow tables

UPDATE radcheck 
SET attribute = 'SHA2-Password', value = SHA2('ваш_пароль', 256) 
WHERE username = 'user';


### Самый стойкий вариант: SHA-512 с солью

mkpasswd -m sha-512 "ваш_пароль"

UPDATE radcheck 
SET Attribute = 'Crypt-Password', Value = '$6$сгенерированный_хеш' 
WHERE UserName = 'user';