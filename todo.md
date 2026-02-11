1. Сделать морду

1.1 Генерация хеша на PHP

<?php
$password = "ваш_пароль";

// Создаем безопасный хеш (bcrypt по умолчанию)
$secureHash = password_hash($password, PASSWORD_BCRYPT);

echo $secureHash;
// Результат будет выглядеть так: $2y$10$abcdefg1234567890.....
?>