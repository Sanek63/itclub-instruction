# SQLAlchemy (Postgres)

## Зависимости
* [SQLAlchemy 1.4.31](https://pypi.org/project/SQLAlchemy/1.4.31/)
* [psycopg2-binary 3.0.8](https://pypi.org/project/psycopg-binary/3.0.8/)

# Инструкция
#### 1) Установить зависимости:
```shell
pip install sqlalchemy==1.4.31 psycopg2-binary=3.0.8
```

#### 2) Проверить установки зависимостей
```shell
pip freeze
```
**Вывод данных должен содержать следующие данные**
```shell
sqlalchemy==1.4.31
psycopg2-binary=3.0.8
```

#### 3) Подключение к базе данных с помощью SQLAlchemy

Для создании сессии необходимо указывать URI адрес базы данных в формате:
```shell
postgresql://{username}:{password}@{host}/{db_name}
```
и произвести подключение с помощью SQLAlchemy. Пример генерации соединения с БД прикладываю ниже

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine(<uri>, pool_pre_ping=True)
Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
session = Session() # Созданная сессия
```

Запустив код выше, указав вместо `<uri>` адрес СУБД, у вас не должно возникать прочих ошибок.
Если у вас возникла ошибка, вчитывайтесь в нее, возможно вы указали неверный адрес URI или у вас не создана
база.

#### 4) Создаем первую сущность
В данном кусочке кода импортирую декоратор `as_declarative` для того, чтобы адаптировать класс Base
под `declarative_base`. Тут описана автоматическая запись названия таблицы в нижний регистр и обязательным
наличием поля id.

В дальнейшем при создании этой сущности, мы наследуем его класс от класса `Base`, тем самым подхватывая атрибуты
`id` и `__tablename__`, которые уже принимают описанное ранее ими значение.

Декоратор `@property` нужен для превращение функции класса в атрибут, чтобы в дальнейшем можно было
обращаться для представления сущности в более простой формат отображения.

На данный момент `el.serialize` вернет нам представление объекта в виде словаря.

Функция `func` из `sqlalchemy.sql` представляет набор встроенных функций в бд, при обращении к которому
позволяет ее сгенерировать. К примеру, текущую дату или количество строк в запрсое.

Функция `Column` позволяет нам описать поле, выбрать для него соответствующие свойства и тип.

К примеру приведенный нижен код создаст целочисленное поле counter, с значением по умолчанию
1, а аттрибут nullable=True не позволяет допускать значение null в текущем поле.
```python
counter = Column(Integer(), default=1, nullable=False)
```

### Создание сущности
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from sqlalchemy import Column, Text, Integer, JSON, BigInteger, TIMESTAMP
from sqlalchemy.sql import func
from sqlalchemy.ext.declarative import as_declarative, declared_attr

engine = create_engine(<uri>, pool_pre_ping=True)
Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
session = Session() # Созданная сессия

@as_declarative()
class Base:
  id: Any
  __name__: str

  @declared_attr
  def __tablename__(cls) -> str:
      return cls.__name__.lower()

class Msg(Base):
  id = Column(BigInteger, primary_key=True, index=True, nullable=False)
  key = Column(Text, nullable=False)
  value = Column(Text, nullable=False)
  counter = Column(Integer, default=1, nullable=False)
  
  created_at = Column(TIMESTAMP, default=func.now(), nullable=False)
  updated_at = Column(TIMESTAMP, default=func.now(), nullable=False, onupdate=func.current_timestamp())
  
  @property
  def serialize(self):
      """Return object data in easily serializable format"""
      return {
          'id': self.id
      }
```

Можно запустить код и убедиться, что все отработает без ошибок.
##### 5) Создадим описанные ранее сущности в нашей БД.
Для создания данной сущности в БД необходимо обратиться к нашему классу `Base`,
который описывает некое представление нашей базы данных (какие сущности и т.п.) и обратиться
к атрибуту `metadata` и у него вызвать функцию `create_all`. В качестве первого атрибута (bind)
передаем наш engine объект.

```python
Base.metadata.create_all(bind=<engine>)
```

Запускаем код и если у вас все отработало без ошибок, заглядываем в базу и смотрим,
что у нас имеется одна таблица msg с описанными ранее атрибутами.

### Примечание
Для описания более одной сущности также создаем отдельные классы, наследуя их от `Base`

```python

class User(Base):
    ...

class Messages(Base):
    ...

class Payment(Base):
    ...
```

и повторяем процедуру создания сущностей в базе тем же способом.

### Что делать если хочу их убрать?
```python
Base.metadata.drop_all(bind=<engine>)
```
вместо 

```python
Base.metadata.create_all(bind=<engine>)
```
