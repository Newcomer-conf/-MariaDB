# Инструкция по установке и настройке MariaDB на Ubuntu, а также по созданию базы данных и добавлению данных.

# Шаг 1: Обновите системные пакеты
```
sudo apt update
```
```
sudo apt upgrade -y
```

# Шаг 2: Установите MariaDB
```
sudo apt install mariadb-server -y
```

# Шаг 3: Запустите и настройте MariaDB
После установки выполните скрипт безопасной настройки:
```
sudo mysql_secure_installation
```

# Вы будете отвечать на ряд вопросов:
Установить пароль для root? Нажмите Enter, если пароль уже установлен.
Удалить анонимных пользователей? Введите Y и нажмите Enter.
Запретить удаленный вход root? Введите Y и нажмите Enter.
Удалить тестовую базу данных и доступ к ней? Введите Y и нажмите Enter.
Перезагрузить таблицы привилегий? Введите Y и нажмите Enter.

# Шаг 4: Создание базы данных и пользователя
## 4.1 Создайте базу данных
Войдите в консоль MariaDB под пользователем root:
```
sudo mariadb
```

Создайте новую базу данных, например, network_db:
```
CREATE DATABASE network_db;
```
# 4.2 Создайте пользователя и предоставьте ему права
Создайте нового пользователя, например, dbuser, и задайте ему пароль:
```
CREATE USER 'dbuser'@'localhost' IDENTIFIED BY 'пароль_пользователя';
```

Предоставьте пользователю все привилегии на базу данных:
```
GRANT ALL PRIVILEGES ON network_db.* TO 'dbuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

# Шаг 5: Создание таблиц и добавление данных
Войдите в базу данных под новым пользователем:
```
mariadb -u dbuser -p
```

Введите пароль при запросе.

Выберите базу данных:
```
USE network_db;
```

# 5.1 Создайте таблицу clients (Таблица 1)

```
CREATE TABLE clients (
  id INT AUTO_INCREMENT PRIMARY KEY,
  ip_address VARCHAR(45) NOT NULL,
  netmask VARCHAR(45) NOT NULL,
  gateway VARCHAR(45) NOT NULL,
  date_created DATETIME NOT NULL,
  date_assigned DATETIME NOT NULL,
  mac_address VARCHAR(17) NOT NULL
);
```

Описание полей:

ip_address: IP-адрес клиента.
netmask: Маска подсети.
gateway: Шлюз по умолчанию.
date_created: Дата создания записи.
date_assigned: Дата назначения IP-адреса.
mac_address: MAC-адрес клиента.

# 5.2 Добавьте данные в таблицу clients
Пример для 10 клиентов:

```
INSERT INTO clients (ip_address, netmask, gateway, date_created, date_assigned, mac_address) VALUES
('10.0.10.1', '255.255.255.0', '10.0.10.254', '2024-10-31 10:00:00', '2024-11-30 10:00:00', 'BC:24:11:00:CC:9F'),
('10.0.10.2', '255.255.255.0', '10.0.10.254', '2024-10-31 11:00:00', '2024-11-30 11:00:00', 'BC:24:11:00:CC:A0'),
('10.0.10.3', '255.255.255.0', '10.0.10.254', '2024-10-31 12:00:00', '2024-11-30 12:00:00', 'BC:24:11:00:CC:A1');
```

# 5.3 Создайте таблицу blocks (Таблица 2)
```
CREATE TABLE blocks (
  id INT AUTO_INCREMENT PRIMARY KEY,
  client_id INT NOT NULL,
  is_blocked BOOLEAN NOT NULL DEFAULT FALSE,
  reason TEXT,
  FOREIGN KEY (client_id) REFERENCES clients(id)
);
```
Описание полей:

client_id: Идентификатор клиента из таблицы clients.
is_blocked: Флаг блокировки (TRUE или FALSE).
reason: Причина блокировки.

# 5.4 Добавьте данные в таблицу blocks

```
INSERT INTO blocks (client_id, is_blocked, reason) VALUES
(1, TRUE, 'Policy violation'),
(2, FALSE, NULL),
(3, FALSE, NULL);
```


# 5.5 Создайте таблицу actions (Таблица 3)

```
CREATE TABLE actions (
  id INT AUTO_INCREMENT PRIMARY KEY,
  client_id INT NOT NULL,
  action_type VARCHAR(50) NOT NULL,
  result VARCHAR(50) NOT NULL,
  open_ports VARCHAR(255),
  event_time DATETIME NOT NULL,
  event_reason TEXT,
  FOREIGN KEY (client_id) REFERENCES clients(id)
);
```

Описание полей:

action_type: Тип действия (например, 'ping', 'tcp', 'udp').
result: Результат действия (например, 'успешно', 'неудачно').
open_ports: Список открытых портов (например, 'HTTP, HTTPS, SSH').
event_time: Время события.
event_reason: Причина события.

# 5.6 Добавьте данные в таблицу actions

```
INSERT INTO actions (client_id, action_type, result, open_ports, event_time, event_reason) VALUES
(1, 'ping', 'success', 'HTTP, SSH', '2024-11-01 09:00:00', 'Availability check'),
(2, 'tcp', 'failure', NULL, '2024-11-01 09:05:00', 'Port closed'),
(3, 'udp', 'success', 'HTTP, HTTPS', '2024-11-01 09:10:00', 'Port scan');
```

# Шаг 6: Проверка данных

Проверьте, что данные добавлены корректно:

Просмотр таблицы clients:
```
SELECT * FROM clients;
```

Результат:
``` 
MariaDB [network_db]> SELECT * FROM clients;
+----+------------+---------------+-------------+---------------------+---------------------+-------------------+
| id | ip_address | netmask       | gateway     | date_created        | date_assigned       | mac_address       |
+----+------------+---------------+-------------+---------------------+---------------------+-------------------+
|  1 | 10.0.10.1  | 255.255.255.0 | 10.0.10.254 | 2024-10-31 10:00:00 | 2024-11-30 10:00:00 | BC:24:11:00:CC:9F |
|  2 | 10.0.10.2  | 255.255.255.0 | 10.0.10.254 | 2024-10-31 11:00:00 | 2024-11-30 11:00:00 | BC:24:11:00:CC:A0 |
|  3 | 10.0.10.3  | 255.255.255.0 | 10.0.10.254 | 2024-10-31 12:00:00 | 2024-11-30 12:00:00 | BC:24:11:00:CC:A1 |
+----+------------+---------------+-------------+---------------------+---------------------+-------------------+
```
------------------------------------------
Просмотр таблицы blocks:

```
SELECT * FROM blocks;
```

Результат:
```
MariaDB [network_db]> SELECT * FROM blocks;
+----+-----------+------------+------------------+
| id | client_id | is_blocked | reason           |
+----+-----------+------------+------------------+
|  1 |         1 |          1 | Policy violation |
|  2 |         2 |          0 | NULL             |
|  3 |         3 |          0 | NULL             |
+----+-----------+------------+------------------+
3 rows in set (0.000 sec)
```
------------------------------------------

Просмотр таблицы actions:
```
SELECT * FROM actions;
```
Результат:
```
MariaDB [network_db]> SELECT * FROM actions;
+----+-----------+-------------+---------+-------------+---------------------+--------------------+
| id | client_id | action_type | result  | open_ports  | event_time          | event_reason       |
+----+-----------+-------------+---------+-------------+---------------------+--------------------+
|  1 |         1 | ping        | success | HTTP, SSH   | 2024-11-01 09:00:00 | Availability check |
|  2 |         2 | tcp         | failure | NULL        | 2024-11-01 09:05:00 | Port closed        |
|  3 |         3 | udp         | success | HTTP, HTTPS | 2024-11-01 09:10:00 | Port scan          |
+----+-----------+-------------+---------+-------------+---------------------+--------------------+
3 rows in set (0.001 sec)
```
------------------------------------------


# Шаг 7: Объединение таблиц с помощью JOIN

7.1. Объединение таблиц clients и blocks
```
SELECT
  clients.id AS client_id,
  clients.ip_address,
  clients.mac_address,
  blocks.is_blocked,
  blocks.reason AS block_reason
FROM
  clients
LEFT JOIN
  blocks ON clients.id = blocks.client_id;
```
Результат:
```

locks    ->    blocks ON clients.id = blocks.client_id;
+-----------+------------+-------------------+------------+------------------+
| client_id | ip_address | mac_address       | is_blocked | block_reason     |
+-----------+------------+-------------------+------------+------------------+
|         1 | 10.0.10.1  | BC:24:11:00:CC:9F |          1 | Policy violation |
|         2 | 10.0.10.2  | BC:24:11:00:CC:A0 |          0 | NULL             |
|         3 | 10.0.10.3  | BC:24:11:00:CC:A1 |          0 | NULL             |
+-----------+------------+-------------------+------------+------------------+
3 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------

Этот запрос выберет всех клиентов и добавит информацию о блокировке из таблицы blocks. Используется LEFT JOIN, чтобы включить всех клиентов, даже если у них нет записи в таблице blocks.

# 7.2. Объединение всех трех таблиц
```
SELECT
  clients.id AS client_id,
  clients.ip_address,
  clients.mac_address,
  blocks.is_blocked,
  blocks.reason AS block_reason,
  actions.action_type,
  actions.result,
  actions.open_ports,
  actions.event_time,
  actions.event_reason
FROM
  clients
LEFT JOIN
  blocks ON clients.id = blocks.client_id
LEFT JOIN
  actions ON clients.id = actions.client_id;
  ```
Результат:
```

+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
| client_id | ip_address | mac_address       | is_blocked | block_reason     | action_type | result  | open_ports  | event_time          | event_reason       |
+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
|         1 | 10.0.10.1  | BC:24:11:00:CC:9F |          1 | Policy violation | ping        | success | HTTP, SSH   | 2024-11-01 09:00:00 | Availability check |
|         2 | 10.0.10.2  | BC:24:11:00:CC:A0 |          0 | NULL             | tcp         | failure | NULL        | 2024-11-01 09:05:00 | Port closed        |
|         3 | 10.0.10.3  | BC:24:11:00:CC:A1 |          0 | NULL             | udp         | success | HTTP, HTTPS | 2024-11-01 09:10:00 | Port scan          |
+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
3 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------

Этот запрос объединяет данные из всех трех таблиц. Вы получите информацию о клиентах, их статусе блокировки и всех связанных действиях.

Примеры использования
Получить информацию о клиентах и их последних действиях
Если вы хотите получить последнее действие каждого клиента, можно использовать подзапрос:

```
SELECT
  c.id AS client_id,
  c.ip_address,
  c.mac_address,
  b.is_blocked,
  b.reason AS block_reason,
  a.action_type,
  a.result,
  a.open_ports,
  a.event_time,
  a.event_reason
FROM
  clients c
LEFT JOIN
  blocks b ON c.id = b.client_id
LEFT JOIN
  actions a ON c.id = a.client_id
WHERE
  a.event_time = (
    SELECT MAX(event_time)
    FROM actions
    WHERE client_id = c.id
  );
```
Результат:
```
+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
| client_id | ip_address | mac_address       | is_blocked | block_reason     | action_type | result  | open_ports  | event_time          | event_reason       |
+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
|         1 | 10.0.10.1  | BC:24:11:00:CC:9F |          1 | Policy violation | ping        | success | HTTP, SSH   | 2024-11-01 09:00:00 | Availability check |
|         2 | 10.0.10.2  | BC:24:11:00:CC:A0 |          0 | NULL             | tcp         | failure | NULL        | 2024-11-01 09:05:00 | Port closed        |
|         3 | 10.0.10.3  | BC:24:11:00:CC:A1 |          0 | NULL             | udp         | success | HTTP, HTTPS | 2024-11-01 09:10:00 | Port scan          |
+-----------+------------+-------------------+------------+------------------+-------------+---------+-------------+---------------------+--------------------+
3 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------


Получить всех заблокированных клиентов с причиной блокировки

```
SELECT
  clients.id AS client_id,
  clients.ip_address,
  clients.mac_address,
  blocks.reason AS block_reason
FROM
  clients
INNER JOIN
  blocks ON clients.id = blocks.client_id
WHERE
  blocks.is_blocked = TRUE;

Получить количество действий для каждого клиента
SELECT
  clients.id AS client_id,
  clients.ip_address,
  COUNT(actions.id) AS action_count
FROM
  clients
LEFT JOIN
  actions ON clients.id = actions.client_id
GROUP BY
  clients.id, clients.ip_address;

```
Результат:
```
+-----------+------------+--------------+
| client_id | ip_address | action_count |
+-----------+------------+--------------+
|         1 | 10.0.10.1  |            1 |
|         2 | 10.0.10.2  |            1 |
|         3 | 10.0.10.3  |            1 |
+-----------+------------+--------------+
3 rows in set (0.000 sec)
```
------------------------------------------------------------------------------------------------------------------

Объяснение операторов JOIN
INNER JOIN: Возвращает записи, у которых есть соответствующие значения в обеих таблицах.
LEFT JOIN: Возвращает все записи из левой таблицы (clients), даже если нет соответствующих записей в правой таблице (blocks или actions).
Советы по объединению данных
Используйте алиасы для таблиц: Это делает запросы более читаемыми.
```
SELECT
  c.id,
  c.ip_address,
  b.is_blocked
FROM
  clients c
LEFT JOIN
  blocks b ON c.id = b.client_id;
```
Результат:
```
+----+------------+------------+
| id | ip_address | is_blocked |
+----+------------+------------+
|  1 | 10.0.10.1  |          1 |
|  2 | 10.0.10.2  |          0 |
|  3 | 10.0.10.3  |          0 |
+----+------------+------------+
3 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------

Фильтрация данных: Добавляйте условия WHERE для отбора только нужных записей.
`
WHERE
  b.is_blocked = TRUE
`
Сортировка результатов: Используйте ORDER BY для сортировки данных.
`
ORDER BY
  c.id ASC
`
Лимитирование результатов: Если нужно получить определенное количество записей, используйте LIMIT.
`
LIMIT 10
`
Пример объединенного запроса с фильтрацией и сортировкой
Получить список клиентов, которые не заблокированы, вместе с их последним действием, отсортированным по времени события:
```
SELECT
  c.id AS client_id,
  c.ip_address,
  c.mac_address,
  a.action_type,
  a.result,
  a.event_time
FROM
  clients c
LEFT JOIN
  blocks b ON c.id = b.client_id
LEFT JOIN
  actions a ON c.id = a.client_id
WHERE
  (b.is_blocked = FALSE OR b.is_blocked IS NULL)
ORDER BY
  a.event_time DESC;
```
Результат:
```
+-----------+------------+-------------------+-------------+---------+---------------------+
| client_id | ip_address | mac_address       | action_type | result  | event_time          |
+-----------+------------+-------------------+-------------+---------+---------------------+
|         3 | 10.0.10.3  | BC:24:11:00:CC:A1 | udp         | success | 2024-11-01 09:10:00 |
|         2 | 10.0.10.2  | BC:24:11:00:CC:A0 | tcp         | failure | 2024-11-01 09:05:00 |
+-----------+------------+-------------------+-------------+---------+---------------------+
2 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------


Дополнительные запросы
Получить список открытых портов для каждого клиента
```
SELECT
  c.id AS client_id,
  c.ip_address,
  a.open_ports
FROM
  clients c
LEFT JOIN
  actions a ON c.id = a.client_id;
```
Результат:
```
+-----------+------------+-------------+
| client_id | ip_address | open_ports  |
+-----------+------------+-------------+
|         1 | 10.0.10.1  | HTTP, SSH   |
|         2 | 10.0.10.2  | NULL        |
|         3 | 10.0.10.3  | HTTP, HTTPS |
+-----------+------------+-------------+
3 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------


Найти клиентов, у которых были неудачные действия
```
SELECT
  c.id AS client_id,
  c.ip_address,
  a.action_type,
  a.result
FROM
  clients c
LEFT JOIN
  actions a ON c.id = a.client_id
WHERE
  a.result = 'success';
```
Результат:
```
+-----------+------------+-------------+---------+
| client_id | ip_address | action_type | result  |
+-----------+------------+-------------+---------+
|         1 | 10.0.10.1  | ping        | success |
|         3 | 10.0.10.3  | udp         | success |
+-----------+------------+-------------+---------+
2 rows in set (0.001 sec)
```
------------------------------------------------------------------------------------------------------------------

 Примечания:

 Замены в шаблоне: Замените пароль_пользователя на ваш собственный пароль.
 Формат дат: Убедитесь, что даты указаны в формате YYYY-MM-DD HH:MM:SS.
 MAC-адреса: MAC-адреса должны быть в формате AA:BB:CC:DD:EE:FF.
 Дополнение данных: Если у вас больше клиентов или другие данные, продолжайте добавлять их в соответствующие таблицы.
 Если нужно создать связи между таблицами:
 В таблице blocks и actions мы используем client_id как внешний ключ, который ссылается на id из таблицы clients. Это связывает записи между собой и облегчает управление данными.

