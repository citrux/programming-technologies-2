## Создание приложения при помощи Django

### 1. Создаём новый проект
В данном примере для управления виртуальными окружениями и зависимостями будет использоваться pipenv. Создаём в текущем каталоге виртуальное окружение, ставим в него Django и активируем его:
```
pipenv install django
pipenv shell
```
Теперь создадим проект под названием example в текущем каталоге:
```
django-admin startproject example .
```
Перед нами возникает набор файлов:
```
├── example
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py
```
* `manage.py`: утилита командной строки для взаимодействия с проектом
* `example/`: каталог проекта
* `example/__init__.py`: чтобы Python рассматривал каталог как пакет
* `example/settings.py`: настройки проекта
* `example/urls.py`: объявления URL
* `example/asgi.py`: entry-point для ASGI-совместимого веб-сервера
* `example/wsgi.py`: entry-point для WSGI-совместимого веб-сервера

Уже сейчас можно запустить development-сервер и увидеть стандартную страницу пустого проекта:
```
python manage.py runserver
```

Проект содержит информацию о конфигурации. Для того, чтобы начать писать код теперь нужно добавить приложение
```
python manage.py startapp shop
```
При этом создаётся каталог shop со следующим содержимым:
```
└── shop
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── tests.py
    └── views.py
```
Теперь всё готово, чтобы заняться приложением.

### 2. Добавим модели

В файл shop/models.py добавим 2 класса моделей:

1. модель товара
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.PositiveIntegerField()
```
2. модель покупки
```python
class Purchase
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    person = models.CharField(max_length=200)
    address = models.CharField(max_length=200)
    date = models.DateTimeField(auto_now_add=True)
```

Чтобы Django смог работать с этими моделями нужно добавить наше приложение shop в список INSTALLED_APPS в файле example/settings.py:
```python
INSTALLED_APPS = [
    'shop.apps.ShopConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
Теперь можно создать миграцию, которая создаст необходимые теблицы в базе данных:
```bash
$ python manage.py makemigrations
Migrations for 'shop':
  shop/migrations/0001_initial.py
    - Create model Product
    - Create model Purchase
```
А затем выполнить её:
```bash
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, shop
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
  Applying shop.0001_initial... OK
```

### 3. Представления, роуты и шаблоны
Теперь создадим нашу первую страницу. Для этого нам потребуется:
1. Написать функцию-представление в shop/views.py
```
from django.shortcuts import render

from .models import Product
# Create your views here.
def index(request):
    products = Product.objects.all()
    context = {'products': products}
    return render(request, 'shop/index.html', context)
```
Здесь shop/index.html -- это расположение шаблона, а context -- словарь с переменными, которые из этого шаблона будут доступны.
2. Создать шаблон shop/templates/shop/index.html
```html
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Товары</title>
</head>
<body>
    <div>
        <h3>Список</h3>
        <table>
            <tr>
                <td><p>Наименование</p></td>
                <td><p>Цена</p></td>
                <td></td>
            </tr>
            {% for p in products %}
                <tr>
                    <td><p>{{ p.name }}</p></td>
                    <td><p>{{ p.price }}</p></td>
                    <td><p><a href="/buy/{{ p.id }}">Купить</a></p></td>
                </tr>
            {% endfor %}
        </table>
    </div>
</body>
</html>
```
3. Создать файл shop/urls.py с привязкой url к функции-представлению
```python
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
4. В файле example/urls.py подключить shop/urls.py к проекту
```python
urlpatterns = [
    path('', include('shop.urls')),
    path('admin/', admin.site.urls),
]
```
Проверим работоспособность, запустив development-сервер:
```bash
$ python manage.py runserver
```
