###Django REST с React: настройка виртуальной среды Python и проект
Прежде всего, убедитесь, что у вас есть виртуальная среда Python . Создайте новую папку и перейдите в нее:

```
mkdir django-react && cd $_
```
После этого создайте и активируйте новую среду Python:

```
python3 -m venv venv
source venv/bin/activate
```
ПРИМЕЧАНИЕ : с этого момента убедитесь, что вы всегда в django-reactпапке и активная среда Python.

Теперь давайте посмотрим на зависимости:

```
pip install django djangorestframework
```
Когда установка закончится, вы готовы создать новый проект Django:

```
django-admin startproject django_react .
```
Теперь мы можем приступить к созданию нашего первого приложения Django: простого API для отображения и хранения контактов .

Django REST с React: создание приложения Django
Проект Django может иметь много приложений . Каждое приложение в идеале должно делать одну вещь. Приложения Django являются модульными и могут использоваться многократно, если другому проекту снова и снова требуется одно и то же приложение, мы можем поместить это приложение в менеджер пакетов Python и установить его оттуда.

Чтобы создать новое приложение в Django, вы должны запустить:

```
django-admin startapp app_name
```
В нашем случае еще в папке проекта запустите:

```
django-admin startapp leads
```
Это создаст наше новое приложение ведет в django-reactпапке. Структура вашего проекта теперь должна быть:

(venv) your@prompt:~/Code/django-react$ tree -d -L 1
```
tree -d -L 1
```
```
.
├── django_react
├── leads
└── venv
```
Теперь давайте расскажем Django, как использовать новое приложение. Откройте django_react/settings.pyи добавьте приложение в INSTALLED_APPS :
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'leads.apps.LeadsConfig', # activate the new app
]
```
Все идет нормально! В следующем разделе мы добавим нашу первую модель.

Django REST с React: создание модели Django
С приложением пришло время создать нашу первую модель. Модель - это объект, представляющий данные вашей таблицы . Почти у каждого веб-фреймворка есть модели, и Django не исключение.

Модель Django может иметь одно или несколько полей: каждое поле является столбцом в вашей таблице. Прежде чем двигаться вперед, давайте определимся с нашими требованиями к ведущему приложению.

Так как я собираю контакты, я могу думать о модели Lead, сделанной из следующих полей:

имя
электронное письмо
сообщение
(Не стесняйтесь добавлять дополнительные поля! Например, телефон). Давайте не будем забывать и поле меток времени! Django по умолчанию не добавляет столбец create_at.

Откройте leads/models.pyи создайте модель Lead:
```
from django.db import models

class Lead(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    message = models.CharField(max_length=300)
    created_at = models.DateTimeField(auto_now_add=True)
```
Краткое примечание о моделях: не торопитесь, чтобы проверить документацию полей Django. При планировании модели постарайтесь выбрать наиболее подходящие поля для вашего варианта использования . И с моделью на месте давайте создадим миграцию, запустив:
```
python manage.py makemigrations leads
```
и, наконец, перенести базу данных с помощью:
```
python manage.py migrate
```
Большой! В следующих разделах мы поговорим о сериализаторах и представлениях . Но сначала заметка о тестировании .

Django REST с React: брызги тестирования
В этот момент вы можете задаться вопросом "Валентино, как насчет тестирования?" Вместо того, чтобы раздражать вас учебником по TDD, я дам вам несколько советов.

Я видел массу учебников по Django, начинающихся так:
```
class SomeModelModelTest(TestCase):
    def setUp(self):
        SomeModel.objects.create(
            name=fake.name(),
            email=fake.email(),
            phone=fake.phone_number(),
            message=fake.text(),
            source=fake.url()
        )
    def test_save_model(self):
        saved_models = SomeModel.objects.count()
        self.assertEqual(saved_models, 2)
```
## Не делай этого. 
##### Нет смысла тестировать ни ванильную модель Django, ни Django ORM . Вот хорошая отправная точка для тестирования в Django:

+ не тестируйте встроенный код Django (модели, представления и т. д.)
+ не тестируйте встроенные функции Python
+ Не проверяйте то, что уже проверено! Так что я должен проверить? Вы добавили пользовательский метод в модель Django? Проверь это! У вас есть пользовательский вид? Проверь это! Но откуда мне знать, что именно тестировать?

Сделай себе одолжение. Установить coverage (покрытие) :
```
pip install coverage
```
Затем каждый раз, когда вы добавляете некоторый код в ваше приложение, запускайте покрытие:
```
coverage run --source='.' manage.py test
```
и сгенерировать отчет:
```
coverage html
```
Вы увидите, что именно нужно проверить . Если вы предпочитаете видеть отчет в командной строке, запустите:
```
coverage report
```
Подожди, ты еще там? Я впечатлен! Держитесь крепче, в следующем разделе мы рассмотрим сериализаторы !

Django REST с помощью React: Django REST сериализаторы
Что такое сериализация? Что такое сериализатор REST Django? Сериализация - это процесс преобразования объекта в другой формат данных. После преобразования объекта мы можем сохранить его в файл или отправить по сети.

Зачем нужна сериализация? Подумайте о модели Django: это класс Python. Как вы отображаете класс Python в JSON в браузере? С сериализатором REST от Django !

Сериализатор работает и наоборот: он преобразует JSON в объекты . Таким образом, вы можете:

отображать модели Django в браузере, конвертируя их в JSON
сделать запрос CRUD с полезной нагрузкой JSON к API
Напомним: сериализатор REST Django является обязательным для работы на моделях через API. Создайте новый файл с именем leads/serializers.py. LeadSerializer использует нашу модель Lead и некоторые поля:
```
from rest_framework import serializers
from .models import Lead

class LeadSerializer(serializers.ModelSerializer):
    class Meta:
        model = Lead
        fields = ('id', 'name', 'email', 'message')
```
Как вы можете видеть, мы подклассифицируем ModelSerializer. ModelSerializer в Django REST похож на ModelForm. Это подходит для случаев, когда вы хотите сопоставить модель с сериализатором.

Сохраните и закройте файл. В следующих разделах мы рассмотрим представления и URL-адреса .

Django REST с React: настройка контроля ... эмм
Исходя из других фреймворков, вы можете удивиться, что у Django нет контроллеров .

Контроллер инкапсулирует логику для обработки запросов и возврата ответов. В традиционной архитектуре MVC есть Модель, Представление и Контроллер. Примером фреймворков MVC являются Rails, Phoenix, Laravel.

Django - это структура MVT . То есть Модель - Вид - Шаблон. В Django существует много типов представлений: представления функций, представления на основе классов и общие представления .

Хотя некоторые разработчики предпочитают функциональные представления вместо представлений на основе классов, я большой поклонник последних . Когда я выбираю Django, это потому, что я ценю скорость разработки, DRY, меньше кода.

Я не вижу смысла в написании представлений вручную, когда уже есть набор нормальных значений по умолчанию. Вот мое эмпирическое правило:

Используйте функциональные представления, только если время, потраченное на настройку общего вида, превышает время, потраченное на написание вида вручную . Как и в простом Django, в среде REST Django есть много способов написания представлений:

функциональные представления
классовые представления
общие представления API
В рамках данного руководства я буду использовать общие представления API . Наше простое приложение должно:

перечислить коллекцию моделей
создавать новые объекты в базе данных
Изучив документацию по общим представлениям API, мы увидим, что существует представление для перечисления и создания моделей: ListCreateAPIView , который принимает набор запросов и serializer_class .

Откройте leads/views.pyи создайте вид:
```
from .models import Lead
from .serializers import LeadSerializer
from rest_framework import generics

class LeadListCreate(generics.ListCreateAPIView):
    queryset = Lead.objects.all()
    serializer_class = LeadSerializer
```
Это. С помощью 3 строк кода мы создали представление для обработки запросов GET и POST . Чего не хватает сейчас? Отображение URL! Другими словами, мы должны сопоставить URL-адреса с представлениями.

Как? Перейдите к следующему разделу.

Django REST с React: настройка маршрута ... URL-адреса
Наша цель - подключить LeadListCreate к api / lead /. Другими словами, мы хотим делать запросы GET и POST к api / lead / для вывода списка и создания моделей.

Чтобы настроить сопоставление URL-адресов, включите URL-адреса приложений в django_react/urls.py:
```
from django.urls import path, include

urlpatterns = [
    path('', include('leads.urls')),
]
```
Затем создайте новый файл с именем leads/urls.py. В этом файле мы подключим LeadListCreate к api / lead /:
```
from django.urls import path
from . import views

urlpatterns = [
    path('api/lead/', views.LeadListCreate.as_view() ),
]
```
Наконец, давайте включим rest_framework в INSTALLED_APPS. Откройте django_react/settings.pyи добавьте приложение в INSTALLED_APPS:
```
# Application definition

INSTALLED_APPS = [
    # omitted for brevity
    'leads.apps.LeadsConfig',
    'rest_framework'
]
```
Теперь вы сможете запустить проверку работоспособности с помощью:
```
python manage.py runserver
```
Зайдите на http://127.0.0.1:8000/api/lead/ и вы увидите API с возможностью просмотра:

Django REST Framework API с возможностью просмотра

ПРИМЕЧАНИЕ. Рекомендуется отключить доступный для просмотра API в рабочей среде с помощью этой конфигурации:
```
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
    )
}
```
Пока что создайте какой-то контакт во встроенной форме . В следующем разделе мы познакомимся с React .

Джанго ОТДЫХ с React: Джанго и Реакт вместе
Многие разработчики Python борются с простым вопросом. Как склеить Django и React вместе?

Маршрутизатор React должен взять на себя маршрутизацию? Должен ли React монтировать компонент в каждом шаблоне Django? (Если хочешь потерять рассудок). Я бы сказал "это зависит" . Это зависит от того, сколько JavaScript вам нужно. Но сколько JavaScript слишком много?

Помимо шуток, существует множество способов настроить проект Django с помощью React. Я вижу следующие шаблоны (которые являются общими почти для всех веб-фреймворков):

Вариант 1. Реагируйте в своем собственном «внешнем» приложении Django: загрузите один HTML-шаблон и позвольте React управлять внешним интерфейсом (сложность: средняя)

Вариант 2. Django REST в качестве отдельного API + Реагирование в качестве отдельного SPA (сложность: сложная, для аутентификации используется JWT)

Вариант 3. Смешивание и сопоставление: мини-приложения React в шаблонах Django (сложность: простая, но в долгосрочной перспективе не слишком удобная для сопровождения)

И вот мои советы. Если вы только начинаете с Django REST и избегаете React, выберите вариант 2. Вместо этого перейдите к варианту № 1 (Реагируйте в своем собственном «внешнем» приложении Django), если:

вы создаете сайт, похожий на приложение
интерфейс имеет много взаимодействий с пользователем / AJAX
вы в порядке с аутентификацией на основе сеанса
нет проблем с SEO
вы в порядке с React Router
На самом деле, поддерживая React ближе к Django, вы сможете легче рассуждать об аутентификации . Вы можете использовать встроенную аутентификацию Django для регистрации и входа в систему пользователей.

Используйте хорошую аутентификацию Session и не беспокойтесь о токенах и JWT.

Выберите вариант № 3 (мини-приложения React внутри шаблонов Django), если:

веб-сайт не нуждается в большом количестве Javascript
SEO - большая проблема, и вы не можете использовать Node.js для рендеринга на стороне сервера
В следующем разделе мы перейдем к варианту 1 .

Настройка React и веб-пакета
Мы уже знаем, как создать приложение Django, поэтому давайте сделаем это снова для приложения внешнего интерфейса :
```
django-admin startapp frontend
```
Вы увидите новый каталог с именем frontend в папке вашего проекта:

(venv) your@prompt:~/Code/django-react$ tree -d -L 1
```
.
├── django_react
├── frontend
├── leads
└── venv
```
Давайте также подготовим структуру каталогов для хранения компонентов React:
```
mkdir -p ./frontend/src/components
```
и статические файлы :
```
mkdir -p ./frontend/{static,templates}/frontend
```
Далее мы настроим React, webpack и babel . Переместите в папку веб-интерфейса и инициализируйте среду:
```
cd ./frontend && npm init -y
```
Далее установите webpack и webpack cli :
```
npm i webpack webpack-cli --save-dev
```
Теперь откройте package.jsonи настройте два сценария, один для производства и один для разработки :
```json
"scripts": {
  "dev": "webpack --mode development ./src/index.js --output ./static/frontend/main.js",
  "build": "webpack --mode production ./src/index.js --output ./static/frontend/main.js"
}
```
Закройте файл и сохраните его. Теперь давайте установим babel для передачи нашего кода :
```
npm i @babel/core babel-loader @babel/preset-env @babel/preset-react --save-dev
```
Следующий шаг в React :
```
npm i react react-dom --save-dev
```
Теперь настройте babel с .babelrc(все еще внутри ./frontend):
```
{
    "presets": [
        "@babel/preset-env", "@babel/preset-react"
    ]
}
```
И, наконец, создайте webpack.config.jsдля настройки babel-загрузчик:
```
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```
Теперь мы готовы к работе! (Добро пожаловать в современный интерфейс!).

Django REST с React: готовим приложение внешнего интерфейса
Перво-наперво создайте представление в ./frontend/views.py:
```python
from django.shortcuts import render


def index(request):
    return render(request, 'frontend/index.html')
```
Затем создайте шаблон в ./frontend/templates/frontend/index.html:
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Django REST with React</title>
</head>
<body>
<div id="app">
    <!-- React will load here -->
</div>
</body>
{% load static %}
<script src="{% static "frontend/main.js" %}"></script>
</html>
```
Как вы можете видеть, шаблон будет вызывать ./frontend/main.js наш пакет веб-пакетов . Сконфигурируйте новое сопоставление URL, чтобы включить внешний интерфейс в ./project/urls.py:
```
urlpatterns = [
    path('', include('leads.urls')),
    path('', include('frontend.urls')),
]
```
Затем создайте новый файл с именем ./frontend/urls.py:
```
from django.urls import path
from . import views


urlpatterns = [
    path('', views.index ),
]
```
Наконец, включите приложение внешнего интерфейса в ./project/settings.py:
```
# Application definition

INSTALLED_APPS = [
    # omitted for brevity
    'leads.apps.LeadsConfig',
    'rest_framework',
    'frontend', # enable the frontend app
]
```
На данный момент вы можете попробовать его на http://127.0.0.1:8000/ (пока работает сервер разработки Django). Сейчас вы увидите пустую страницу .

В следующем разделе мы наконец добавим React к миксу .

Джанго ОТДЫХ с React: интерфейс React
Для простоты мы создадим простой компонент React, который будет отображать наши данные . Если у вас ничего нет в базе данных, это хороший момент, чтобы заполнить ваше приложение каким-либо контактом .

Запустите сервер разработки и перейдите по ссылке http://127.0.0.1:8000/api/lead/, чтобы вставить несколько потенциальных клиентов.

Теперь создайте новый файл в ./frontend/src/components/App.js. Это будет компонент React для извлечения и отображения данных:
```
import React, { Component } from "react";
import { render } from "react-dom";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      data: [],
      loaded: false,
      placeholder: "Loading"
    };
  }

  componentDidMount() {
    fetch("api/lead")
      .then(response => {
        if (response.status > 400) {
          return this.setState(() => {
            return { placeholder: "Something went wrong!" };
          });
        }
        return response.json();
      })
      .then(data => {
        this.setState(() => {
          return {
            data,
            loaded: true
          };
        });
      });
  }

  render() {
    return (
      <ul>
        {this.state.data.map(contact => {
          return (
            <li key={contact.id}>
              {contact.name} - {contact.email}
            </li>
          );
        })}
      </ul>
    );
  }
}

export default App;

const container = document.getElementById("app");
render(<App />, container);
```
ПРИМЕЧАНИЕ : вы можете написать тот же компонент в виде функции с помощью useEffect ловушки.

Сохраните и закройте файл. Теперь создайте точку входа для WebPack в ./frontend/src/index.jsи импортировать компонент:
```
import App from "./components/App";
```
Теперь мы готовы проверить вещи . Запустите веб-пакет с:
```
npm run dev
```
Запустите сервер разработки:
```
python manage.py runserver
```
и над головой http://127.0.0.1:8000/ . (Если вы видите «Что-то пошло не так», обязательно перенесите и заполните свою базу данных)

Наконец вы должны увидеть свои данные в компоненте React:

Простой компонент React