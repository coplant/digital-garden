---
  tags:
    - Basic
    - Python
    - OOP
---

# _Объектно-ориентированное программирование_
## Объекты и классы

```py
class ClassName(object):
    attribute = "value"

exm = ClassName()
dir(ClassName)  # все атрибуты класса
dir(exm)  # все атрибуты экземпляра

print(id(exm.attribute))  # 2258757320560
print(id(ClassName.attribute))  # 2258757320560
```

Изначально `id(exm.attribute) == id(ClassName.attribute)`. Для того, чтобы экономить память Python привязывает экземпляр к тому же месту в памяти. Если же изменить атрибут внутри класса, то он изменится во всех экземплярах, так как они ссылаются на один адрес. Если изменить значение внутри экземпляра, то атрибут экземпляра получит новый адрес и при изменении атрибута самого класса, атрибут экземпляра изменяться больше не будет.

```python
exm.attribute = "newValue"
print(id(exm.attribute))  # 2258758287920
print(id(ClassName.attribute))  # 2258757320560
```

Так же если `id(exm.attribute) == id(ClassName.attribute)`, то `ClassName.__dict__` выведет все атрибуты класса, а `exm.__dict__` - пустой словарь:

```python
print(ClassName.__dict__)  # {'__module__': '__main__', 'attribute': 'value', '__dict__': <attribute '__dict__' of 'ClassName' objects>, '__weakref__': <attribute '__weakref__' of 'ClassName' objects>, '__doc__': None}
print(exm.__dict__)  # {}
```

Это происходит потому что `exm` ссылается на все данные из `ClassName`. Если мы изменим атрибут в экземпляре `exm`, то выведется только новое значение атрибута:

```python
print(exm.__dict__)  # {'attribute': 'newValue'}
```

---

## Атрибуты (`setattr, getattr, delattr, hasattr`)

```python
class User:
    name = ""
    surname = ""

if __name__ == '__main__':
    user = User()
    user.name = "Ivan"
    user.surname = "Noir"
```

### `setattr(object, name, value)`

`object` - объект, значение атрибута которого требуется получить,  
`name` - имя атрибута, должно быть строкой,  
`value` - произвольное значение атрибута.

Устанавливает значение атрибута, если атрибута не существует, то создается новый атрибут с именем `name`

```python
print("New (before):", getattr(user, "new", None))
setattr(user, "new", "NewAttribute")
print("New (after):", getattr(user, "new", None))
```

### `getattr(object, name, default)`

`object` - объект, значение атрибута которого требуется получить,  
`name` - имя атрибута, должно быть строкой,  
`default` - значение по умолчанию, которое будет возвращено, если имя атрибута `name` отсутствует.

Возвращает значение именованного атрибута объекта.

```python
print(getattr(user, "name", None))
```

Эквивалент `print(user.name)`, но:

- полезно когда изначально неизвестно какой атрибут возвращать
- когда точно неизвестно есть ли у атрибута значение ()

### `delattr(object, name)`

`object` - объект, значение атрибута которого требуется получить,  
`name` - имя атрибута, должно быть строкой.

Удаляет указанный атрибут (если он существует!)

```python
delattr(user, "new")
print(getattr(user, "new", None))  # None
```

### `hasattr(object, name)`

`object` - объект, значение атрибута которого требуется получить,  
`name` - имя атрибута, должно быть строкой.

Проверяет существование атрибута

```python
 print(hasattr(user, 'new'))
```

[Атрибуты setattr, getattr, delattr, dict](https://www.notion.so/setattr-getattr-delattr-dict-116a070e4d054698a0d5e9ba98f19224?pvs=21) 

---

## Инициализация объектов и аргументов `self`

При явном указании переменной и её значения, эта переменная принадлежит классу. Если создать экземпляр, но не изменить значение переменной, то они все будут ссылаться на одну и ту же ячейку в памяти ([Объекты и классы](index.md) ), тем самым при изменении переменной она изменится во всех экземплярах. Для этого придумали инициализацию.

```python
class User:
    def __init__(self, *args, **kwargs):
        self.args = args
        self.kwargs = kwargs
```

При создании объекта сначала вызывается метод `__new__`, а затем метод `__init__`. 

Таким образом, при создании объекта переменные будут находиться внутри экземпляра, и при изменении переменной в классе, экземпляры изменяться не будут. Для экономии памяти одинаковые переменные ссылаются на одну ячейку памяти.

Так же использование `self` необязательно, вместо этого можно использовать любое название. Однако это не рекомендуется, поскольку код будет сложнее понять.  

```python
class User:
    def __init__(this, *args, **kwargs):
        this.args = args
        this.kwargs = kwargs
```

В методе `__init__` можно вызывать другие методы.

```python
class User:
    def __init__(self, name):
        self.name = name
        self.get_name()

    def get_name(self):
        print(self.name)
```

[Инициализация __init__](https://www.notion.so/__init__-a765f058d88c4a3bb1f48d39b0d89fb0?pvs=21) 

---

## Свойства (`getter, setter, deleter`)

Мы можем получать и изменять атрибуты обращаясь к ним напрямую, но мы не контролируем их запись и чтение, что небезопасно. Для того, чтобы это было безопасно и удобно можно использовать геттеры и сеттеры.

```python
class User:
    def __init__(self, name):
        self.name = name

    def get_name(self):
        return self.name

    def set_name(self, value):
        self.name = value
```

Но данный код все еще небезопасный. Во-первых, необходимо задать нашему атрибуту приватность. Нижнее подчеркивание вначале делает атрибут приватным. Так же можно задать значение по-умолчанию в `__init__`:

```python
def __init__(self, name=None):
        self._name = name
```

Но текущая запись все еще небезопасна. Для того, чтобы сделать ее более безопасной используется свойство `@property`. Метод внутри `@property` доступен только для чтения.

```python
@property
def get_name(self):
    return self._name

@get_name.setter  # используем название метода выше
def set_name(self, name):
    self._name = name
```

Так же вместо `get_name` можно использовать `name`. Конечный вид класса:

```python
class User:
    def __init__(self, name=None):
        self._name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, name):
        self._name = name

    @name.deleter
    def name(self):
        del self._name
```

[Свойства getter, setter, deleter](https://www.notion.so/getter-setter-deleter-a750a5ffea8541ac93a5c00718f4573b?pvs=21) 

---

## Dunder методы (`get, set`)

При использовании [Свойства getter, setter, deleter](https://www.notion.so/getter-setter-deleter-a750a5ffea8541ac93a5c00718f4573b?pvs=21) для каждого атрибута, класс быстро наполняется ненужным кодом, тем самым усложняя его чтение. Для того, чтобы такого не было, были созданы дескрипторы.

Для начала создадим класс дескриптора:

```python
class Data:
    def __init__(self):
        self.value = None

    def __get__(self, instance, owner):
        return self.value

    def __set__(self, instance, value):
        self.value = value
```

Затем создаем класс `User`, и создаем два атрибута:

```python
class User:
    name = Data()
    surname = Data()
```

При создании атрибута вызывается метод `__init__` класса `Data` и создается `self.value` с пустым значением.

Создаем экземпляр класса `User` (тем самым внутри класса создаются экземпляры `name` и `surname` от класса `Data`)

```python
if __name__ == '__main__':
    user = User()
    user.name = "Ivan"
    user.surname = "Noir"
    print(user.name, user.surname)
```

В таком случае нам не нужно делать свойства для каждого атрибута, мы можем создать класс `Data`, включающий в себя методы `__get__` и `__set__`.

[Методы __get__, __set__](https://www.notion.so/__get__-__set__-41d45576e29a409da08c34c081faf94f?pvs=21) 

---

## Статические методы (`@staticmethod`)

Статические методы позволяют использовать методы класса напрямую без создания экземпляра класса.

Создадим класс `User`:

```python
class User:
    def __init__(self, name="", age=20):
        self.name = name
        self.age = age

    def get_data(self):
        print(self.name)
        print(self.age)

    @staticmethod
    def get_sum(x, y):
        print(x + y)
```

При вызове метода `User.get_data()` напрямую возникает ошибка:

```python
Traceback (most recent call last):
  File "E:\Python Projects\OOP\static.py", line 17, in <module>
    User.get_data()
TypeError: User.get_data() missing 1 required positional argument: 'self'
```

Но мы можем вызвать метод `User.get_sum(1, 2)` напрямую, который является статическим.

Используя статические методы (`@staticmethod`) мы можем вызывать методы напрямую без создания экземпляров.

Используя статический метод (`@staticmethod`), метод больше не является методом самого экземпляра, а является функцией. Статические методы используются чтобы явно указать к какому объекту мы принадлежим.

---

## `__slots__` - оптимизируем потребление памяти

Атрибут `__slots__` позволяет ограничить атрибуты, явно указав те, которые мы используем; сокращает потребление памяти.

Создадим класс и его экземпляр:

```python
class User:
    def __init__(self, name="", age=20):
        self.name = name
        self.age = age

if __name__ == '__main__':
    user = User()
```

Мы можем получить значения `name` и `age`, так же мы можем создавать атрибуты напрямую, используя экземпляр:

```python
    user.attr = "value"
```

Добавим атрибут `__slots__` в класс. Перечислим все используемые атрибуты:

```python
class User:
    __slots__ = ["name", "age"]
    def __init__(self, name="", age=20):
        self.name = name
        self.age = age
```

При попытке создания нового атрибуты возникает ошибка:

```python
Traceback (most recent call last):
  File "E:\Python Projects\OOP\slots.py", line 10, in <module>
    user.attr = "value"
AttributeError: 'User' object has no attribute 'attr'
```

Если заранее указать атрибут в `__slots__`, то его можно будет создавать напрямую, используя экземпляр. Помимо этого, добавлять названия методов в  `__slots__` не нужно. 

Если `__slots__` наследуется в другой класс, то его необходимо так же объявлять в каждом классе отдельно (наследование класса не наследует  `__slots__`). 

---

## Dunder метод (`new`)

Метод `__new__` вызывается самый первый при создании экземпляра класса, после него вызывается метод `__init__`.

`def __new__(cls, *args, **kwargs):  
cls` - принимаемый в качестве объекта класс (аналог `self`, но для класса).  
`*args, **kwargs` - параметры, которые обязательно используются при вызове `__new__`. 

Создадим класс:

```python
class Test:
    def __new__(cls, *args, **kwargs):
        print("Creating instance of class")
        instance = super().__new__(cls)
        # instance = super(Test, cls).__new__(cls)  # Старая версия
        return instance
```

В методе `super` принимаем `Test` (родительский класс) и ссылку на сам класс. Вызываем dunder-метод `__new__`, используя класс. Возвращаем `instance`, тем самым у нас создается экземпляр.

Таким образом, мы можем изменять поведение нашего класса даже при создании экземпляра.

Так же мы можем добавить метод `__init__`:

```python
    def __init__(self, name):
        print("Attributes initialization!")
        self.name = name
```

Мы можем использовать метод `__new__` для обхода `__init__`, иногда бывает полезно пропустить инициализацию атрибутов:

```python
class Test:
    def __init__(self, name):
        print("Attributes initialization!")
        self.name = name

if __name__ == '__main__':
    a = Test.__new__(Test)
```

---

## Методы класса (`@classmethod`)

Декоратор `@classmethod` позволяет привязать метод к самому классу и вызывать его от класса. В случае использования `@classmethod` мы модифицируем сам класс.

Для начала создадим простой класс и несколько методов:

```python
class User:
    version = 1

    def __init__(self, name="noname"):
        self.name = name

    @classmethod
    def set_name(cls, value):
        cls.version = value
```

Декоратор `@classmethod` позволяет изменить переменную во всех экземплярах, которые создаются из этого класса (изменяет глобально в классе). Если попытаться изменить это значение напрямую, то мы изменим его только в одном экземпляре.

```python
if __name__ == '__main__':
    a = User()
    b = User()
    c = User()
    a.set_name(2)
    b.version = 3
    print(a.version)  # 2
    print(b.version)  # 3
    print(c.version)  # 2
```

Если мы явно укажем значение атрибута в экземпляре, то он перестанет ссылаться на ту же ячейку в памяти, что и сам класс. Таким образом, при изменении атрибута экземпляра (`b.version = 3`), он начинает ссылаться на другую ячейку (тем самым не ссылаясь на класс).  

Так же мы можем использовать декоратор `@classmethod` для того чтобы вернуть новый экземпляр:

```python
    @classmethod
    def set_name(cls, value):
        return cls(value)
```

В таком случае мы обращаемся к классу и передаем `value`. Когда мы будем возвращать класс, мы попадем в dunder-метод `__init__`, тем самым создадим новый экземпляр и проинициализируем атрибуты.

```python
if __name__ == '__main__':
    a = User.set_name("A Name")
    print(a.name)  # "A Name"
```

Чтобы воспользоваться методом с `@classmethod` нам необязательно создавать экземпляр класса. 

Если же мы создаем экземпляр, обращаемся к `.set_name()` и передаем аргументы, мы получаем экземпляр.

```python
if __name__ == '__main__':
    a = User.set_name("A Name")
    b = a.set_name("B Name")
    print(b.name)  # B Name
```

В итоге при помощи класс-метода мы можем обращаться к атрибутам класса, так же мы можем создавать экземпляры внутри класса и внутри экземпляра. `@classmethod` принимает аргумент класса (`cls`), поэтому ему доступны все атрибуты, которые присутствуют в самом классе.

---

## Инкапсуляция - публичные, приватные и защищенные атрибуты

Создадим класс:

```python
class User:
    def __init__(self, name=""):
        self.name = name
```

В данном случае атрибут `name` публичный. Если создать экземпляр класса, то мы спокойно можем обратиться к данному атрибуту. Для того, чтобы сделать атрибут приватным, нужно поставить одно подчеркивание вначале (`_`).

```python
class User:
    def __init__(self, name=""):
        self._name = name
```

Данный атрибут все еще можно прочитать или изменить. Но в Python принято, что если используется приватный атрибут, то доступ к такому атрибуту получать не нужно.

Для того, чтобы сделать наш атрибут защищенным, нам нужно использовать два нижних подчеркивания (`__`). В таком случае доступ к нашему атрибуту уже будет невозможно получить напрямую.

```python
class User:
    def __init__(self, name=""):
        self.__name = name
```

В `__dict__` такой атрибут записывается с названием класса (`_User__name`).

```python
if __name__ == '__main__':
    a = User("A")
    print(a.__dict__)  # {'_User__name': 'A'}
```

Однако мы все еще можем получить и модифицировать атрибут при помощи `a._User__name`. То есть любую инкапсуляцию можно обойти. Никакой как таковой инкапсуляции у нас нет.

```python
    a._User__name = "New Name"
    print(a._User__name)  # "New Name"
```

Определим атрибут `__slots__`.

```python
class User:
    __slots__ = ["__name"]
    def __init__(self, name=""):
        self.__name = name
```

С помощью следующего подхода мы не можем вызвать `__dict__`. Однако если мы попытаемся вызвать `a._User__name`, то у нас все получится. То есть в Python есть только лишь сокрытие данных, которое держится на словах: если разработчик указал одно или два нижних подчеркивания, то мы понимаем, что это приватный атрибут и доступ к нему получать не нужно.

---

## Моносостояние - принцип работы

Создадим класс:

```python
class User:
    args = {
        "version": 1,
        "flags": 2
    }

    def __init__(self):
        self.__dict__ = self.args
```

При обращении в `__init__` мы обращаемся к нашему экземпляру, вызываем метод `__dict__` (т. е. получаем все атрибуты), и записываем содержимое `args` в его атрибуты. Когда мы передаем ссылку, то мы ссылаемся на одно и то же значение в памяти. Если изменить это значение внутри класса, то это изменится во всех экземплярах. Изменение любого экземпляра изменит все экземпляры, в том числе и класс.

```python
if __name__ == '__main__':
    a = User()
    b = User()
    c = User()
    print(a.__dict__)  # {'version': 1, 'flags': 2}
    print(a.args)  # {'version': 1, 'flags': 2}
```

Попробуем изменить любое значение нашего словаря:

```python
    a.args["version"] = 2  
    print(a.args)  # {'version': 2, 'flags': 2}
```

Значение изменилось в экземпляре `a`, так же оно изменилось в экземплярах `b` и `c`.

```python
    print(b.args)  # {'version': 2, 'flags': 2}
    print(c.args)  # {'version': 2, 'flags': 2}
```

Кроме того оно так же изменилось в самом классе, поскольку мы ссылаемся на ту же ячейку в памяти, тем самым любое изменение изменяет ее глобально.

```python
    print(User.args)  # {'version': 2, 'flags': 2}
```

---

## Полиморфизм на примере (`@singledispatch`)

```python
from functools import singledispatch
```

Полиморфизм (в Python) похож на перегрузку методов. Например, когда мы наследуем класс, внутри которого есть метод с таким же названием, и мы внутри дочернего класса реализуем метод с таким же именем. Т. е. создаем тот же метод внутри дочернего класса, который есть в родительском классе. Так же помимо полиморфизма присутствует перегрузка операторов, dunder-методов.

Полиморфизм в компилируемых ЯП - один метод может обрабатывать различные типы данных.

При помощи декоратора `@singledispatch` мы можем создавать методы, поведение которых будет отличаться от различных типов данных.

Создадим класс:

```python
from functools import singledispatch

class User:
    @singledispatch
    def get_value(value):
        print("default:", value)

    @get_value.register(int)
    def _(value):
        print("int:", value)

    @get_value.register(str)
    def _(value):
        print("str:", value)
```

При попытке передать неописанный тип данных мы попадаем в `default`. Во всех остальных случаях поведение будет зависеть от передаваемого типа данных. 

```python
if __name__ == '__main__':
    User.get_value(0)  # int: 0
    User.get_value("null")  # str: null
    User.get_value(None)  # default: None
```

[Полиморфизм `@singledispatch`](https://www.notion.so/singledispatch-9bc3e844970f46a7856cdfb7e938045d?pvs=21) 

---

## Dunder методы (`str, repr, len, del`)

Переопределение dunder-методов.

```python
class User:
    def __init__(self, value="null"):
        self.value = value

    def __len__(self):
        return len(self.value)

    def __str__(self):
        return self.value

    def __repr__(self):  # в отличии от __str__, 
        return f"<{self.value} object>"  # в __repr__ находится отладочная информация

    def __del__(self):
        print("Instance deleted!")
```

```python
if __name__ == '__main__':
    a = User()
    print(len(a))  # 4
    print(str(a))  # null
    print(repr(a))  # <null 0x27e3d92dab0 object>
    del a  # "Instance deleted!", затем экземпляр удаляется
```

`print(a)` автоматически преобразует экземпляр к `__str__`.

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Dunder методы (`bool, bytes, float, int`)

Dunder-методы для работы с типами данных.

```python
class User:
    def __init__(self, value="null"):
        self.value = value

    def __bool__(self):
        return bool(self.value)

    def __bytes__(self):
        return self.value.encode()

    def __int__(self):
        return int(self.value)
```

```python
if __name__ == '__main__':
    a = User(3.1) 
    print(bool(a))  # True
    print(bytes(a))  # b'\x00\x00\x00'
    print(int(a))  # 3
    print(float(a))  # 3.1
```

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Dunder методы (`pow, reversed, truediv`)

Dunder-методы для работы с операторами

```python
class User:
    def __init__(self, value):
        self.value = value
        self.arrage = [1, 2, 3]

    def __pow__(self, power, modulo=None):
        return self.value ** power

    def __reversed__(self):
        self.arrage.reverse()
        return self.arrage

    def __truediv__(self, other):
        return self.value // other
```

```python
if __name__ == '__main__':
    a = User(10)
    print(a ** 2)  # 100
    print(reversed(a))  # [3, 2, 1]
    print(a / 5)  # 2
```

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Dunder методы (`next, iter, call`)

```python
class User:
    def __init__(self, value):
        self.value = value

    def __call__(self, comment):  # метод позволяет вызывать экземпляр как функцию
        print("__call__:", self.value)
        print("comment:", comment)

    def __iter__(self):
        print("__iter__")
        self.arrage = self.data.copy()
        return self

    def __next__(self):
        if self.arrage:
            return f"__next__: {self.arrage.pop()}"
        else:
            raise StopIteration("No elements!")
```

Вызов экземпляра как функции:

```python
a("help")  # __call__: 10 \n comment: help
```

Вызов итератора:

```python
for i in a:
    print(i)
```

`__iter__` вызывается с самого начала, инициализирует сам итератор. Возвращаем ссылку на наш экземпляр.  
`__next__` вызывается каждый раз при новой итерации.

```python
__iter__
__next__: 3
__next__: 2
__next__: 1
```

Так же мы можем итерироваться сами без помощи цикла. Для этого инициализируем итератор:

```python
iter(a)
next(a)  # __next__: 3
next(a)  # __next__: 2
next(a)  # __next__: 1
next(a)  # StopIteration: No elements!
```

Если мы заново вызовем метод `__iter__`, то мы переместимся в начало списка.

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Контекстный менеджер в классе используя `enter, exit`

```python
class User:
    def __init__(self):
        self.arrage = [1, 2, 3]

    def __enter__(self):
        print("__enter__")
        return self.arrage

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("__exit__")
        del self.arrage

if __name__ == '__main__':
    a = User()
    with a as array:
        print(array[0])
```

При вызове контекстного менеджера возвращаемое значение из `__enter__` попадает в `array`. После того, как мы вышли из контекстного менеджера (в том числе при возникновении ошибки) автоматически вызывается метод `__exit__`.

`with` часто используется для открытия файлов. В конце работы с файлом его обязательно нужно закрыть, `with` делает это все автоматически.

[Контекстный менеджер в классе](https://www.notion.so/cf46b9fb269740e099edd75aa6a96aae?pvs=21) 

---

## Dunder методы (`add, sub, mul, eq, hash`)

Dunder-методы для работы с операторами. Перегрузка операторов

```python
class User:
    def __init__(self, value):
        self.value = value

    def __add__(self, other):
        return self.value + other

    def __sub__(self, other):
        return self.value - other

    def __mul__(self, value):
        return self.value * other

    def __eq__(self, other):
        if other == self.value:
            return True
        else:
            return False
```

```python
    def __hash__(self):
        return hash(self.value)

    def __eq__(self, other):
        return isinstance(other, User) and self.value == other.value
```

```python
if __name__ == '__main__':
    print(User(1) + 2)  # 3
    print(User(1) - 2)  # -1
    print(User(2) * 2)  # 4
    print(hash(User(0)) == hash(User("Hello")))  # False
    print(hash(User(1337)) == hash(User(1337)))  # True
```

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Dunder методы (`getitem, setitem, delitem`)

```python
class User:
    def __init__(self, value=[1, 2, 3]):
        self.data = value

    def __getitem__(self, item):
        return self.data[item]

    def __setitem__(self, key, value):
        self.data[key] = value

    def __delitem__(self, key):
        del self.data[key] 
```

```python
if __name__ == '__main__':
    a = User(0)
    print(a[0])  # 1
    a[0] = 5
    print(a[0])  # 5
    del a[0]
    print(a[0])  # 2
```

[Переопределение dunder-методов](https://www.notion.so/dunder-8174f2ab3e544682b1ba70fbb6999168?pvs=21) 

---

## Наследование и перегрузка методов

```python
class ClassName(ParentClass):
```

Для того, чтобы избавиться от повторяющегося кода в ООП присутствует принцип наследования. Очень часто классы имеют схожие атрибуты или поведение, которые могут быть вынесены в отдельный базовый класс (родительский класс). Классы, наследуемые от базового класса называются подклассами. 

```python
class Wizzard:
    def go_to(self):
        print("go_to")

class Fighter:
    def go_to(self):
        print("go_to")
```

```python
class Base:
    def go_to(self):
        print("go_to")

class Wizzard(Base):
    pass

class Fighter(Base):
    pass
```

При наследовании подкласс приобретает все атрибуты и методы родительского класса. Они добавляются к собственным методам и атрибутам подкласса.

Так же бывает такое, когда некоторые метода необходимо переопределить. 

```python
class Base:
    def go_to(self):
        print("go_to")

    def get_damaged(self):
        print("get_damaged")

    def restore_health(self):
        print("restore_health")

class Wizzard(Base):
    def get_damaged(self):  # переопределение метода .get_damaged()
        print("None")

class Fighter(Base):
    pass

class Paladin(Base):
    pass
```

При наследовании в дочерний клаас, `__init__` класса родителя не вызывается.

```python
class Base:
    def __init__(self, health):
        self.health = health

class Wizzard(Base):
    def __init__(self, health=100):
        super().__init__(health)
        # Base.__init__(self, health)

class Paladin(Base):
    def __init__(self, health=200):
        super().__init__(health)
```

Метод `super()` берет по умолчанию класс родителя (то есть обращаемся к родительским методам). Когда мы используем родительский класс напрямую, то нам нужно указывать `self` (то есть `super().__init__(health) == Base.__init__(self, health)`).

[Наследование и перегрузка](https://www.notion.so/e84a2c1cf26b40aa8b4d9a51765e259c?pvs=21) 

---

## `isinstance, issubclass, getsizeof`

### `issubclass(class, classinfo)`

`class` - класс (не экземпляр!), требующий проверки,  
`classinfo` - класс или [к](https://docs-python.ru/tutorial/osnovnye-vstroennye-tipy-python/tip-dannyh-tuple-kortezh/)ортеж с классами. C Python 3.10, несколько классов могут записываться как побитовое или - `|`

Позволяет определить ссылается ли наш дочерний класс на определенный родительский класс.

```python
class Paladin(Base):
    def __init__(self, health=200):
        super().__init__(health)
```

```python
issubclass(Paladin, Base)  # True
```

### `sys.getsizeof(object[, default])`

`object` - объект Python,  
`default` - [`int`](https://docs-python.ru/tutorial/osnovnye-vstroennye-tipy-python/tip-dannyh-int-tselye-chisla/), значение по умолчанию.

Возвращает размер объекта `object` в байтах. Учитывается только потребление памяти, непосредственно приписываемое объекту, а не потребление памяти объектами, к которым он относится.

```python
import sys
f = Fighter()
print(sys.getsizeof(Fighter()))  # 48, т.к Fighter ссылается на Base
```

Учитывается только потребление памяти, непосредственно приписываемое объекту, а не потребление памяти объектами, к которым он относится.

```python
print(sys.getsizeof(Fighter))  # 1070
```

### `isinstance(object, classinfo)`

`object` - объект, требующий проверки,  
`classinfo` - класс, кортеж с классами или рекурсивный кортеж кортежей. С версии Python 3.10 может быть объединением нескольких типов (например `int | str`).

Функция вернет `True`, если проверяемый объект `object` является экземпляром указанного класса (классов) или его подкласса (прямого, косвенного или виртуального).

Если объект `object` не является экземпляром данного типа, то функция всегда возвращает `False`.

```python
print(isinstance("string", str))  # True
```

---

## Переопределение методов родителя

### **`__getattr__(self, name)`**

Метод `__getattr__` служит для того, чтобы отлавливать несуществующие атрибуты.

```python
class Test:
    def __getattr__(self, item):
        print("No such method!")
```

Запрашивается атрибут, если указанного атрибута не существует, то вызывается метод `__getattr__`.

### `setattr(object, name, value)`

```python
class Base:
    _fields = []

    def __init__(self, *args):
        for name, value in zip(self._fields, args):
            setattr(self, name, value)
```

Когда мы используем `__init__` у нас принимается `*args`, затем мы используем `zip()` чтобы соединить `self._fields` и `args`. `self._fields` - список наших атрибутов. Используя функцию `setattr()` мы обращаемся к `self` и задаем `name` = `value`

```python
class Stock(Base):
    _fields = ["name", "shares", "price"]

a = Stock(1, 2, 3)
a.__dict__  # {"name": 1, "shares": 2, "price": 3}
```

Как это все устроено: сначала мы создаем экземпляр класса и передаем в него параметры. Так же в дочернем классе создаем список `_fields` с названиями атрибута. Так как мы наследуем класс родителя, то первым делом мы попадаем в него для того, чтобы вызвать метод `__init__`. В методе `__init__` при помощи функции `zip()` сопоставляем значения из `self._fields` и `args`, при помощи функции `setattr(self, name, value)` устанавливаем атрибуты.

### **`__getattribute__(self, name)`**

Используется для того, чтобы явно указывать какой атрибут мы вызываем.

```python
class Base:
    def __getattribute__(self, item):
        print("getting:", item)
        return super().__getattribute__(item)
```

```python
class Stock(Base):
    def __init__(self, x):
        self.x = x

    def spam(self):
        pass

s = Stock(0)
s.x  # "getting: x"
s.spam()  # "getting: spam"
```

Поскольку мы наследуем класс, нам нужно обращаться к методу `super().__getattribute__(item)`

### Переопределение готовых структур

```python
class MyDict(dict):
    def __setitem__(self, key, value):
        print("__setitem__")
        super().__setitem__(key, value * 3)
        # dict.__setitem__(self, key, value * 3)

    def __getitem__(self, item):
        return super(MyDict, self).__getitem__(item) == 1

d = MyDict(one=1, two=2)
print(d)  # {'one': 1, 'two': 2}
d["two"] = 3
print(d)  # __setitem__ \n {'one': 1, 'two': 9}
print(d["one"])  # True 
```

 [Переопределение готовых структур](https://www.notion.so/dae5531f3e884f3ba11cc933e94ee69c?pvs=21) 

---

## Дата-классы (`@dataclass`)

Дата-классы хранят в себе значения и очень похожи на структуры в других языках. Они нужны для хранения определенных данных.

Для того, чтобы использовать дата-классы используются два модуля:

```python
from typing import Any  # для того, чтобы задавать тип данных нашим значениям
from dataclasses import dataclass  # сам декоратор
```

Создадим дата-класс. Для того, чтобы каждый раз не указывать значения, мы можем инициализировать значения по-умолчанию.

```python
@dataclass
class User:
    name: str = "no name"
    age: int = -1
    flags: list = None
    comment: Any = None
```

Далее мы можем создать экземпляр класса и передать следующие значения:

```python
a = User(
    name="Name",
    age=20,
    flags=[True, False],
    comment="?!"
)

```

В итоге мы можем обратиться к любому значению:

```python
print(a)  # User(name='Name', age=20, flags=[True, False], comment='?!')
print(a.name)  # "Name"
```

Наследование дата-классов осуществляется как и с обычными классами:

```python
class Test(User):
    def get_name(self):
        return self.name
```

Все наследуемые из дата-класса атрибуты находятся в экземпляре класса `Test`.

[Дата-классы @dataclass](https://www.notion.so/dataclass-c5dd4c43462d4a45af835b9d4da498a8?pvs=21) 

---

## Множественное наследование (`mro, vars, callable, super`)

### `vars(object)`

`object` - объект.

Возвращает атрибут `dict` объекта.

```python
class Info:
    verion = "1.0"

    def get_version(self):
        return self.verion

print(vars(Info))  # {'__module__': '__main__', 'verion': '1.0', 'get_version': <function Info.get_version at 0x000002974D83E830>, '__dict__': <attribute '__dict__' of 'Info' objects>, '__weakref__': <attribute '__weakref__' of 'Info' objects>, '__doc__': None}
```

Мы можем проверить является ли атрибут вызываемы при помощи функции `callable()`:

```python
for key, value in vars(Info).items():
    if callable(value):
        print(f"{key} is callable!")
    else:
        print(f"{key} is NOT callable!")
```

```
__module__ is NOT callable!
verion is NOT callable!
get_version is callable!
__dict__ is NOT callable!
__weakref__ is NOT callable!
__doc__ is NOT callable!
```

### Множественное наследование (`mro`)

Создадим класс. Класс `Template` наследуется от классов `Test` и `Test2`.

```python
class Test:
    @staticmethod
    def get():
        print("Test")

class Test2:
    @staticmethod
    def get():
        print("Test2")

class Template(Test, Test2):
    def __init__(self, name):
        self.name = name
```

`Template.mro()` показывает все наследование данного объекта слева  направо.

```python
# print(Template.__mro__)
print(Template.mro())  # [<class '__main__.Template'>, <class '__main__.Test'>, <class '__main__.Test2'>, <class 'object'>]
```

`.mro()` нужен для того, чтобы понять как нужно наследовать сами классы. Допустим в классе `Test` и `Test2` есть методы с одинаковыми названиями, но с разным поведением. В таком случае основному классу нужно понять какой метод вызвать первым. То есть, при обращении к методу с одинаковым названием, основной класс обращается к `.mro()` и смотрит какой класс идет первым. 

```python
t = Template("name")
t.get()  # "Test"
```

Таким образом, нужно запомнить, что мы используем наследование слева направо, то есть: сначала основной класс, затем его родители, после сам объект. 

Для того, чтобы контролировать вызов методов, мы можем обращаться к родителям напрямую.

```python
class Template(Test, Test2):
    def __init__(self, name):
        self.name = name

    @staticmethod
    def start():
        Test.get()  # "Test"
        Test2.get()  # "Test2"
```

Используя данный синтаксис мы можем обращаться ко всем методам, даже если их названия будут совпадать.
Метод `super()` требует аргумент `self`, так, как когда мы вызываем метод из экземпляра, то автоматически добавляется `self` в качестве первого аргумента.

```python
class Test:
    def get(self):
        print("Test")

class Test2:
    def get(self):
        print("Test2")

class Template(Test, Test2):
    def __init__(self, name):
        self.name = name
        super().get()
```

Прямой вызов `super()` использует первый наследуемый класс.

Реализуем инициализацию внутри каждого родителя:

```python
class Test:
    def __init__(self, name):
        self.name = name

class Test2:
    def __init__(self, age):
        self.age = age

class Template(Test, Test2):
    def __init__(self, version):
        self.version = version
```

Внутри экземпляра отсутствуют атрибуты `name` и `age`, так, как мы не вызывали инициализацию внутри родительских классов.

```python
a = Template(0)
print(vars(a))  # {'version': 0} 
```

Попробуем обратиться к `super()`:

```python
class Template(Test, Test2):
    def __init__(self, version):
        self.version = version
        super().__init__("name")
```

В итоге получаем `version` и `name`, так как мы использовали `super()` и `__init__`:

```python
a = Template(0)
print(vars(a))  # {'version': 0, 'name': 'name'} 
```

Для того, чтобы инициализировать `Test2`, нам нужно обращаться к родителям напрямую (при этом нам нужно передавать атрибут `self`):

```python
class Template(Test, Test2):
    def __init__(self, version):
        self.version = version
        Test.__init__(self, "name")
        Test2.__init__(self, 20)
```

Если родительские методы обращаются к одному и тому же атрибуту, то в дочернем классе они перезапишутся.

---

## Абстрактные методы (`ABCMeta, abstractmethod`)

```python
from abc import ABCMeta, abstractmethod
```

Создадим абстрактные методы. В класс `User` передаем метакласс `ABCMeta`:

```python
from abc import ABCMeta, abstractmethod

class User(metaclass=ABCMeta):
    @abstractmethod
    def go_to(self):
        pass

    @abstractmethod
    def read(self):
        pass
```

В абстрактных классах нам не нужно расписывать поведение методов, они нужны только для создания интерфейса.

Создадим класс, наследующий класс `User`:

```python
class Test(User):
    pass
```

При создании экземпляра класса `Test` мы получим ошибку.

```
TypeError: Can't instantiate abstract class Test with abstract methods go_to, read
```

Когда мы наследуем абстрактный класс, мы говорим дочернему классу, что ему нужно реализовать методы, описанные в абстрактном классе. Если мы не реализуем данные методы, то экземпляр дочернего класса просто не будет создан.

```python
class Test(User):
    def go_to(self):
        print("goto")

    def read(self):
        print("read")
```

Таким образом, основной класс заставляет дочерние классы реализовывать все указанные методы.

---

## Декораторы классов

Создадим класс-декоратор:

```python
class Counter:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)

    def say(self, comment):
        print(comment)
```

 Создадим декорируемую функцию:

```python
@Counter
def printer():
    print("printer")
```

Изначально у нас есть адрес нашей функции, далее этот адрес попадает в декоратор `@Counter`, тем самым инициализируя этот адрес в `__init__`. То есть теперь мы можем обращаться к нашей функции без ее вызова, и вызывать методы класса `Counter`. При вызове функции вызывается метод `__call__`.

```python
printer.say("hello")  # hello
print(printer.count)  # 0
printer()
printer()
print(printer.count)  # 2
```

---

## Динамическое редактирование класса

Создадим класс:

```python
class Test:
    pass

def get_version(self):
    return 22

Test.version = get_version
```

В итоге в класс `Test` добавился метод `version`, который использует адрес функции `get_version()`

```python
print(vars(Test))  # {'__module__': '__main__', '__dict__': <attribute '__dict__' of 'Test' objects>, '__weakref__': <attribute '__weakref__' of 'Test' objects>, '__doc__': None, 'version': <function get_version at 0x000002222D843F40>}
print(Test.version)  # <function get_version at 0x000002222D843F40>
```

Создадим экземпляр и попробуем вывести нашу функцию:

```python
a = Test()
print(vars(a))  # {}, т.к функция находится в самом классе
print(a.version)  # 22
```

Попробуем динамически определять атрибуты:

```python
class Test:
    def __init__(self):
        self.__dict__["version"] = 1

a = Test()
print(vars(a))  # {'version': 1}
```

Сначала мы обращаемся к `__dict__`, далее обращаемся к ключу (т. е. создаем новый ключ `version` со значением `1`). Так же обращаясь к ключу мы можем изменить его значение.

```python
a.__dict__["new"] = 123
print(a.new)  # 123
```

Обращаясь напрямую к классу мы не можем задавать нужное нам значение при помощи `__dict__`, но можем задавать значение напрямую обращаясь к нашим атрибутам:

```python
Test.__dict__["new"] = 123  # TypeError: 'mappingproxy' object does not support item assignment
Test.new = 123  # успешно
```

В экземпляре мы можем изменять значения через `__dict__` поскольку это больше не `mappingproxy`, который нельзя изменять, а обычный словарь.

```python
import sys

class ChangeMe:
    def __init__(self):
        local = sys._getframe(1).f_locals
        self.__dict__.update(
            (key, value) for key, value in local.items() if callable(value))

def test1():
    print("test1")

def test2():
    print("test2")

a = ChangeMe()
a.test1()  # test1
a.test2()  # test2
```

В данном случае мы объявляем функции `test1` и `test2`. Хоть у нас и нет никакой связи с классом, но мы можем вызвать эти функции из нашего экземпляра. Все дело в том, что переменная `local` получает все локальные атрибуты. Далее используя `self.__dict__.update()` мы обновляем словарь нашего экземпляра, если значение из словаря `local` вызываемое.

```python
print(local)  # {'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <_frozen_importlib_external.SourceFileLoader object at 0x000001ED785C8850>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, '__file__': 'E:\\Python Projects\\OOP\\dynamically edition.py', '__cached__': None, 'sys': <module 'sys' (built-in)>, 'ChangeMe': <class '__main__.ChangeMe'>, 'test1': <function test1 at 0x000001ED78193F40>, 'test2': <function test2 at 0x000001ED7861E710>}
```

---

## Метаклассы

Создадим класс:

```python
class Test:
    pass

a = Test()
```

При помощи скобок мы создаем экземпляр класса. Метаклассы - технология, которая создает сами классы. То есть - метакласс создает класс, а класс создает экземпляр. Когда мы пишем `class Test:`, вызывается метакласс, который создает объект класса.

Функция `type()` так же является метаклассом. При помощи нее мы можем создавать классы:

`type(name, base, attr)`  
`name` - имя класса,  
`()` - кортеж родительских классов,  
`{}` - словарь, содержащий атрибуты и их значения.

```python
classObject = type("className", (), {})
print(classObject)  # <class '__main__.className'>
instance = classObject()
print(instance)  # <__main__.className object at 0x000001B04F25FA30>
```

Таким образом, мы создали класс `classObject` и экземпляр класса `instance`.

Попробуем динамически определить атрибуты, и при помощи функции `type()` создать класс:

```python
def get_version():
    return 1

cls_dict = {'version': get_version, 'name': "name", 'age': 0}
new_cls = type("Meta", (), cls_dict)
print(vars(new_cls))  # {'version': <function get_version at 0x000002C8B19A3F40>, 'name': 'name', 'age': 0, '__module__': '__main__', '__dict__': <attribute '__dict__' of 'Meta' objects>, '__weakref__': <attribute '__weakref__' of 'Meta' objects>, '__doc__': None}
```

```python
print(new_cls.name)  # name
a = new_cls()
print(a.name)  # name
```

Создадим класс, запрещающий создавать экземпляры:

```python
class NoInstances(type):
    def __call__(self, *args, **kwargs):
        raise TypeError("Can not create instance!")

class Test(metaclass=NoInstances):
    @staticmethod
    def test():
        print("test")
```

При попытке создать экземпляр класса мы получим ошибку, однако мы можем обращаться напрямую к классу:

```python
a = Test()  # TypeError: Can not create instance!
Test.test  # test
```

При создании экземпляра мы вызываем наш класс при помощи круглых скобок (`()`). В качестве метакласса наследуем  `metaclass=NoInstances`. Класс `NoInstances` определил, что это вызываемая функция и мы попадаем в dunder-метод `__call__`. Метод `__call__` отрабатывает каждый раз, когда мы создаем экземпляр класса.

Синглтон - класс, который позволяет создать только один экземпляр.

Попробуем реализовать синглтон:

```python
class Singleton(type):
    def __init__(self, *args, **kwargs):
        self.__instance = None
        super().__init__(*args, **kwargs)

    def __call__(self, *args, **kwargs):
        if self.__instance is None:
            self.__instance = super(Singleton, self).__call__(*args, **kwargs)
        else:
            return self.__instance

class Test(metaclass=Singleton):
    def __init__(self):
        print("Instance created")

a = Test()  # "Instance created"
b = Test()  # экземпляр не создался
```

[Метаклассы](https://www.notion.so/14cd38cd3028484e982982070babf724?pvs=21) 

---
