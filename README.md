# first django


# help for beginer

### Cài đặt Django

Tốt nhất là bạn nên dùng virtualenv, và pip để cài Django.

```
$ sudo apt-get update && sudo apt-get install python-virtualenv python-pip build-essential python-dev libffi-dev libssl-dev libxml2-dev libxslt1-dev postgresql python-psycopg2 -y
$ mkdir myfirstapp
$ cd myfirstapp
$ virtualenv -p python3 djangotut
$ djangotut/bin/pip install django

source djangotut/bin/activat

python3 -m django --version

django-admin startproject mysite

python3 manage.py runserver

python3 manage.py runserver 8080

python3 manage.py runserver 0.0.0.0:8000

python3 manage.py startapp polls

<code file='polls/views.py'>

from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

</code>

<code file='polls/urls.py'>


from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]

</code>

<code file='mysite/urls.py'>

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]

</code>

python3 manage.py runserver

# Go to http://localhost:8000/polls/

python3 manage.py migrate

<code file='polls/models.py'>

from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

</code>

<code file='mysite/settings.py'>

INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

</code>

python3 manage.py makemigrations polls

python3 manage.py sqlmigrate polls 0001

python3 manage.py migrate

# vao portgredb kiem tra 

python manage.py shell

>>> from polls.models import Question, Choice   # Import the model classes we just wrote.
>>> Question.objects.all()
[]

>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

>>> q.save()

>>> q.id
1

>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

>>> q.question_text = "What's up?"
>>> q.save()

>>> Question.objects.all()
[<Question: Question object>]

<code file'polls/models.py'>

import datetime

from django.db import models
from django.utils import timezone

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text

</code>

>>> from polls.models import Question, Choice

>>> Question.objects.all()
[<Question: What's up?>]

>>> Question.objects.filter(id=1)
[<Question: What's up?>]
>>> Question.objects.filter(question_text__startswith='What')
[<Question: What's up?>]

>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

>>> Question.objects.get(id=2)

>>> Question.objects.get(pk=1)
<Question: What's up?>


>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True


>>> q = Question.objects.get(pk=1)


>>> q.choice_set.all()
[]


>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)


>>> c.question
<Question: What's up?>


>>> q.choice_set.all()
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]
>>> q.choice_set.count()
3


>>> Choice.objects.filter(question__pub_date__year=current_year)
[<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]


>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()

# create account admin
python manage.py createsuperuser

# password: admin123123

<code file='polls/admin.py'>

from django.contrib import admin

# Register your models here.
from .models import Choice, Question

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question,QuestionAdmin)
admin.site.register(Choice)

</code>

<code file='polls/views.py'>

def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)

</code>

<code file='polls.urls'>

from django.conf.urls import url

from . import views

app_name = 'polls'
urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]

</code>

<code file='polls/views'>

from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged

</code>

<code file='polls/templates/polls/index.html'>

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

</code>

<code file='polls/views.py'>

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

</code>

<code file='polls/views.py'>

from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)

</code>

<code file='polls/views.py'>

from django.http import Http404
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    # question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})

</code>

<code file='polls/templates/polls/detail.html'>


<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>

</code>
<code file='polls/templates/polls/detail.html'>

<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="/polls/{{ question.id }}/vote/" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>

</code>

<code file='polls/views.py'>

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))

</code>

<code file='polls/views.py'>

from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})

</code>

<code file='polls/templates/polls/results.html'>

<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="/polls/{{ question.id }}/">Vote again?</a>

</code>

# run test
python3 manage.py runserver

<code file='polls/urls.py'>

from django.conf.urls import url

from . import views

urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]

</code>

<code file='polls/views.py'>

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.

</code>

# tich hop django-suit de customize giao dien admin
pip install django-suit==0.2.25

# hoac them 'django-suit==0.2.25' vao file requitement.txt

<code file='mysite/settings.py'>

INSTALLED_APPS = (
    ...
    'suit',
    'django.contrib.admin',
)

</code>

<code file='mysite/settings.py'>

# Django Suit configuration example
SUIT_CONFIG = {
    # header
    'ADMIN_NAME': 'Heliosys Administrator',
    
    'HEADER_DATE_FORMAT': 'l, j. F Y',
    'HEADER_TIME_FORMAT': 'H:i',

    # menu
    'SEARCH_URL': '/admin/auth/user/',
    'MENU_ICONS': {
        'sites': 'icon-leaf',
        'auth': 'icon-lock',
    },
    # 'MENU_EXCLUDE': ('auth.group',),
    'MENU': (
        # 'sites',
        # {'app': 'auth', 'icon':'icon-lock', 'models': ('user', 'group')},
        {'label': 'Settings', 'icon':'icon-cog', 'models': ('auth.user', 'auth.group')},
        {'label': 'Support', 'icon':'icon-question-sign', 'url': '/support/'},
        {'label': 'polls', 'icon':'icon-question-sign','app':'polls', 'models': (
            {'model': 'question', 'label': 'Question'},
            {'model': 'choice', 'label': 'Choice'}
        )},
    ),

    # misc
    'LIST_PER_PAGE': 15
}

</code>

```
