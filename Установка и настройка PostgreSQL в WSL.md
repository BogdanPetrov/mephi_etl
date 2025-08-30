# Установка и настройка PostgreSQL в WSL

Установите WSL (подробная инструкция: https://learn.microsoft.com/ru-ru/windows/wsl/install)

```
wsl --install
```

Запустите WSL

```
wsl -d Ubuntu
```

Установите PostgreSQL

```
sudo apt-get update
sudo apt-get install postgresql-16 libpq-dev
```

Создайте БД

```
sudo -u postgres createdb etl
```

Подключитесь к БД через `psql`

```
sudo -u postgres psql
```

Создайте пользователя `etl` и выдайте права

```
CREATE ROLE etl;
ALTER ROLE etl WITH PASSWORD 'etl';
ALTER ROLE etl WITH LOGIN;
GRANT ALL ON DATABASE etl TO etl;
```

Настройте доступ к созданной БД 

```
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Добавьте строчку в конец файла

```
host    etl     etl     0.0.0.0/0       password
```

и сохраните файл последовательно нажав клавиши `Ctrl+X`, `y`, `Enter`. 

Настройте PostgreSQL, чтобы к БД можно было подключиться снаружи

```
sudo nano /etc/postgresql/16/main/postgresql.conf
```

Найдите строчку с параметром `listen_addresses`, расскоментируйте (нужно удалить `#` в начале строчки) и выставьте значение параметра следующим образом

```
listen_addresses = '*'
```

Сохраните файл последовательно нажав клавиши `Ctrl+X`, `y`, `Enter`. 

Теперь необходимо перезапустить сервер PostgreSQL для применения новых параметров командой

```
sudo systemctl restart postgresql
```

Проверьте подключение к БД из Python

```python
from sqlalchemy import create_engine
import polars as pl
DATABASE_URL = 'postgresql+psycopg2://etl:etl@localhost:5432/etl'
db = create_engine(DATABASE_URL)
q = """SELECT 1"""
pl.read_database(q, db)
```

```
shape: (1, 1)
?column?
i64
1
```
