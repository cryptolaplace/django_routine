# django_routine
the fastest way to build django webserver

## Get database running
If you are developing a small project or something you don’t plan to deploy in a production environment, SQLite is generally the best option as it doesn’t require running a separate server. 

## Install Django
python -m pip install Django
> verifying: 
>> import django
>> print(django.get_version()) 

## Creating Virtual Env
python3 -m venv ~/.virturalenvs/djangodev
source ~/.virturalenvs/djangodev/bin/activate

## Creating a project
django-admin startproject mysite

## Files intro
> The outer mysite/ root directory is a container for your project. Its name doesn’t matter to Django; you can rename it to anything you like.

> manage.py: A command-line utility that lets you interact with this Django project in various ways. 

> The inner mysite/ directory is the actual Python package for your project. Its name is the Python package name you’ll need to use to import anything inside it (e.g. mysite.urls).

> mysite/__init__.py: An empty file that tells Python that this directory should be considered a Python package. 

> mysite/settings.py: Settings/configuration for this Django project. 

> mysite/urls.py: The URL declarations for this Django project; a “table of contents” of your Django-powered site. 

> mysite/asgi.py: An entry-point for ASGI-compatible web servers to serve your project.

> mysite/wsgi.py: An entry-point for WSGI-compatible web servers to serve your project. 

## The dev server
python manage.py runserver
python manage.py runserver 8080
python manage.py runserver 0:8000
> 0 is short for 0.0.0.0

## Creating app
python manage.py startapp polls

## view
### polls/view.py

>from django.http import HttpResponse  
>def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

### polls/urls.py
> from django.urls import path  
> from . import views  
>urlpatterns = [path('',views.index,name='index'),]

### mysite/urls.py
> from django.contrib import admin  
> from django.urls import include, path  
> urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]

## Database setup
python manage.py migrate

## Creating models
polls/models.py

## Activating models
To include the app in our project, we need to add a reference to its configuration class in the INSTALLED_APPS setting. The PollsConfig class is in the polls/apps.py file, so its dotted path is 'polls.apps.PollsConfig'. Edit the mysite/settings.py file and add that dotted path to the INSTALLED_APPS setting.
> python manage.py makemigrations polls
You can read the migration for your new model if you like; it’s the file polls/migrations/0001_initial.py. 
There’s a command that will run the migrations for you and manage your database schema automatically - that’s called migrate, and we’ll come to it in a moment - but first, let’s see what SQL that migration would run. The sqlmigrate command takes migration names and returns their SQL
> python manage.py sqlmigrate polls 0001
The sqlmigrate command takes migration names and returns their SQL:
> python manage.py sqlmigrate polls 0001
The sqlmigrate command doesn’t actually run the migration on your database - instead, it prints it to the screen so that you can see what SQL Django thinks is required. It’s useful for checking what Django is going to do or if you have database administrators who require SQL scripts for changes.
If you’re interested, you can also run python manage.py check; this checks for any problems in your project without making migrations or touching the database.

Remember the three-step guide to making model changes:
1. Change your models (in models.py)  
2. Run python manage.py makemigrations to create migrations for those changes  
3. Run python manage.py migrate to apply those changes to the database.

## Playing with the API
> python manage.py shell  

We’re using this instead of simply typing “python”, because manage.py sets the DJANGO_SETTINGS_MODULE environment variable, which gives Django the Python import path to your mysite/settings.py file.

> from polls.models import Choice, Question  
Question.objects.all()  
from django.utils import timezone  
q = Question(question_text="What's new?", pub_date=timezone.now())  
q.save()

> adding a __str__() method to models helps dealing with the objects.all() return like <QuerySet [<Question: Question object (1)>]>

> common command  
Question.objects.get(pk=1)  
q = Question.objects.get(pk=1)  
q.was_published_recently()  
q.choice_set.all()  
q.choice_set.create(choice_text='Not much', votes=0)  
q.choice_set.create(choice_text='The sky', votes=0)  
c = q.choice_set.create(choice_text='Just hacking again', votes=0)  
c.question  
q.choice_set.all()  
q.choice_set.count()  
Choice.objects.filter(question__pub_date__year=current_year)  
c = q.choice_set.filter(choice_text__startswith='Just hacking')  
c.delete()  

## Creating an admin user
python manage.py createsuperuser

## make the poll app modifiable in the admin
polls/admin.py
from django.contrib import admin
from .models import Question
admin.site.register(Question)

## writing more views
polls/views.py¶
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)
def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)
def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)

polls/urls.py¶
from django.urls import path
from . import views
urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]

## write views that actually do something
polls/views.py
from django.http import HttpResponse
from .models import Question
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

> template system  
First, create a directory called templates in your polls directory. Django will look for templates in there.
Your project’s TEMPLATES setting describes how Django will load and render templates. The default settings file configures a DjangoTemplates backend whose APP_DIRS option is set to True. By convention DjangoTemplates looks for a “templates” subdirectory in each of the INSTALLED_APPS.
Within the templates directory you have just created, create another directory called polls, and within that create a file called index.html. In other words, your template should be at polls/templates/polls/index.html. Because of how the app_directories template loader works as described above, you can refer to this template within Django as polls/index.html.

> polls/templates/polls/index.html  
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

> polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))


## Raising a 404 error

> polls/views.py
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})

> polls/templates/polls/detail.html
{{ question }}


### A shortcut: get_object_or_404()

> polls/views.py
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

## Use the template system
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>

## Removing hardcoded URLs in templates¶
> <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>  
--> <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>

## Namespacing URL names
polls/urls.py
> app_name = 'polls'
change your polls/index.html
{% url 'polls:detail' question.id %}

