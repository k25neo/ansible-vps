# ansible-vps

Запуск контейнера с терминалом:
docker compose run --rm ansible

## Запуск апгрейда:
ansible-playbook -i hosts playbook-debian-12-to-13.yml

## Установка openConnect:
ansible-playbook -i hosts playbook-openconnect.yml

Для проверки используйте --check


## Создать пользователя
ocpasswd -c /etc/ocserv/ocpasswd username.
