# Celery

## Зависимости
* [Celery 5.2.3](https://pypi.org/project/celery/5.2.3/)
* [Celery redis](https://pypi.org/project/celery/5.2.3/)
* [Redis](https://redis.io/download)


# Инструкция по установке Redis
## Debian Linux
#### 1) Обновить локальные пакеты
```shell
sudo apt update
sudo apt upgrade
```
#### 2) Установка Redis
```shell
sudo apt install redis-server
```

#### 3) С помощью текстового редактора, например vi, откройте конфигурационный файл, который генерируется автоматически:
```shell
sudo vi /etc/redis/redis.conf
```
С помощью поиска найдите параметр supervised. Значение параметра указывает на систему инициализации, по умолчанию имеет значение no, необходимо заменить это значение на systemd:
`supervised systemd`

#### 4) Перезагрузите СУБД:
```shell
sudo systemctl restart redis.service
```

#### 5) Проверка Redis
Для того, чтобы убедиться, что сервер работает, выполните следующую команду:
```shell
sudo systemctl status redis
```

Результат:
```shell
redis-server.service - Advanced key-value store
   Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor pre
   Active: active (running) since Thu 2018-10-11 14:31:06 MSK; 33min ago
     Docs: http://redis.io/documentation,
           man:redis-server(1)
  Process: 23557 ExecStop=/bin/kill -s TERM $MAINPID (code=exited, status=0/SUCC
  Process: 23561 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exi
 Main PID: 23581 (redis-server)
    Tasks: 4 (limit: 4915)
   CGroup: /system.slice/redis-server.service
           └─23581 /usr/bin/redis-server 127.0.0.1:6379

Oct 11 14:31:06 Ubuntu1804x64 systemd[1]: Starting Advanced key-value store...
Oct 11 14:31:06 Ubuntu1804x64 systemd[1]: redis-server.service: Can't open PID f
Oct 11 14:31:06 Ubuntu1804x64 systemd[1]: Started Advanced key-value store.

```

#### 6) Чтобы проверить корректность работы Redis, подключитесь к серверу с помощью клиента командной строки:
```shell
redis-cli
```

#### 7) Проверьте соединение с помощью команды ping:
```shell
127.0.0.1:6379> ping
PONG
```

#### 8) Результат PONG подтверждает, что соединение с сервером установлено. Затем убедитесь, что установка ключей Redis доступна:
```shell
127.0.0.1:6379> set test "1cloud"
OK
```

#### 9) Теперь получите заданное значение, также после перезапуска сервера значение должно сохраниться:
```shell
127.0.0.1:6379> get test
1cloud
```

#### 10) Для выхода из клиента используйте сочетание клавиш Ctrl+С или команду:
```shell
127.0.0.1:6379> exit
```

##GNOME Linux
#### 1) Установка redis
```shell
$ sudo pacman -S redis
```

#### 2) Запускаем сервер
```shell
$ sudo redis-server
```

#### 3) Проверяем с помощью ping через CLI:
```shell
$ redis-cli ping
PONG
```

## Windows
TODO

# Инструкция по установке Celery Linux
#### 1) Установить зависимости:
```shell
pip install celery=5.2.3 celery[redis]
```

Установка пакета `celery[redis]` нужна для поддержки работы с Redis.
Это как зависимость, без нее не будет работать celery

#### 2) Создаем файл `celery_app.py` с содержимым
```python
from celery import Celery

app = Celery(
    'name_queue', broker='redis://{ip}:{port}/0'
) # Первый атрибут имя очереди. Второй URI адрес Redis сервера

app.conf.imports = app.conf.imports + (
    "tasks",
) # Подключаем питоновский модуль с тасками
```

#### 3) Создаем файл tasks.py с содержимым
```python
from main import app
from time import sleep

@app.task
def test():
    sleep(5)
    return 'test sleep 5 seconds'
```

В данном файле мы создали функцию тест, в которой вызывается только функция `sleep` с задержкой в 5 секунд.


#### 4) Запускаем наш celery worker сервер
```shell
celery -A celery_app worker -l INFO --concurrency=100 
```
В данной команде по тегу `-A` указываем модуль, где находится наше celery приложение. По тегу
`-l` указываем уровень логирования (INFO) и в `--concurrency` мы указываем количество
рабочих процессов/потоков для многопроцессорности и одновременного выполнения задач. Количество задается
количеством ядер на устройстве.

Также обратите внимание на раздел `tasks` в консольном выводе. Там должны отображаться
ваши таски в модуле tasks.py

#### 4) Создаем файл main.py с содержимым:
```python
from tasks import test

test.delay()
```

#### 5) Запускаем файл main.py.
В окне, где запущен у вас celery сервер, вы должны увидеть информацию о том. Что получили
задачу и через 5 (т.к. задержка в 5 секунд) секунд должна появиться информация о том,
что данная задача успешно завершена и с каким результатом. Если такой информации не имеется,
проверьте соединение с Redis или же проверьте в конфигурациях, подключили ли вы модуль к нашему приложению.

Возможен такой случай, что ваше приложение просто не видит ваши таски и поэтому их не выполняет.


## Хочу много функций
В модуле tasks.py продолжайте описывать ваши функции. При запуске приложения, если вы
подключили данный модуль, текущие функции будут подхвачены и приложение будет их видеть.
```python
@app.task
def send_sms():
    return 

@app.task
def send_email():
    return 
```
