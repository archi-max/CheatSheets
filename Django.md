# Django

# Setting up

### Creating a Project

```
django-admin startproject mysite
```

### Creating an APP

1. `python manage.py startapp poll`
2. In settings, add to Installed apps  `'appname.apps.AppnameConfig'`  [For models]

### Routing

mysite/urls.py

```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

polls/urls.py

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
```

# Creating the website

### Models

```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Updating the app to add the changes in model to SQL

1. `python manage.py makemigrations polls`
2. `python manage.py sqlmigrate polls 0001`
3. `python manage.py migrate`

### Creating superuser

```
python manage.py createsuperuser
```

### Admin File

In `admin.py`

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

### Render

```python
render(request, 'polls/index.html', context)
```

### Forms

```python
from django import forms

class NameForm(forms.Form):
    your_name = forms.CharField(label='Your name', max_length=100)
```

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import NameForm

def get_name(request):
    # if this is a POST request we need to process the form data
    if request.method == 'POST':
        # create a form instance and populate it with data from the request:
        form = NameForm(request.POST)
        # check whether it's valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required
            # ...
            # redirect to a new URL:
            return HttpResponseRedirect('/thanks/')

    # if a GET (or any other method) we'll create a blank form
    else:
        form = NameForm()

    return render(request, 'name.html', {'form': form})
```

```python
<form action="/your-name/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```

looping over form fields

```python
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
        {% if field.help_text %}
        <p class="help">{{ field.help_text|safe }}</p>
        {% endif %}
    </div>
{% endfor %}
```

Model Form

```python
class AuthorForm(ModelForm):
    class Meta:
        model = Author
        fields = ['name', 'title', 'birth_date']
```

```python
fields = '__all__'
```

You can use `exclude` instead of fields to exclude the fields

```python
exclude = ['name', 'title']
```

For saving the form details to a model : `form.save(commit=True)`

For cleaning a field

```python
from django import forms
from django.core.exceptions import ValidationError

class ContactForm(forms.Form):
    # Everything as before.
    ...

    def clean_recipients(self):
        data = self.cleaned_data['recipients']
        if "fred@example.com" not in data:
            raise ValidationError("You have forgotten about Fred!")

        # Always return a value to use as the new cleaned data, even if
        # this method didn't change it.
        return data
```

For cleaning main

```python
from django import forms

class ContactForm(forms.Form):
    # Everything as before.
    ...

    def clean(self):
        cleaned_data = super().clean()
        cc_myself = cleaned_data.get("cc_myself")
        subject = cleaned_data.get("subject")

        if cc_myself and subject and "help" not in subject:
            msg = "Must put 'help' in subject when cc'ing yourself."
            self.add_error('cc_myself', msg)
            self.add_error('subject', msg)
```

Initial Data

```python
f = ContactForm(initial={'subject': 'Hi there!'})
```

Using uploaded files

```python
<form enctype="multipart/form-data" method="post" action="/foo/">
```

while creating a form with files

```python
f = ContactFormWithMugshot(request.POST, request.FILES) 
```

widgets represent the html part.

### Static Files

First, create a directory called **`static`** in your **`polls`** directory. Django will look for static files there, similarly to how Django finds templates inside **`polls/templates/`**.

Django’s [STATICFILES_FINDERS](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-STATICFILES_FINDERS) setting contains a list of finders that know how to discover static files from various sources. One of the defaults is **`AppDirectoriesFinder`** which looks for a “static” subdirectory in each of the **[INSTALLED_APPS](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-INSTALLED_APPS)**

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

### Templates

[Built-in template tags and filters | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/ref/templates/builtins/#ref-templates-builtins-filters)

various variables

```html
{{ var }} # var
{ my_dict.key }} #dictionary
{{ my_object.attribute }} #class
{{ my_list.0 }} #list

{{ django|title }} - Title Case
{{ my_date|date:"Y-m-d" }} - Date Formatting
{{ value|default:"nothing" }} - Default
{% cycle 'row1' 'row2' as rowcolors silent %} - Cycles values. 
{% firstof var1 var2 var3 "fallback value" %} - first value which is not false
{% now "jS F Y H:i" %} - Current data with formatting
{{ value|linenumbers }}  - displays each line\element in value with the line number
{{ num_messages|pluralize }} - adds s if value is not 1
{% include "foo/bar.html" %} - replaces tag with the html code in file

```

loops, control statements

```python
{% if athlete_list %}
    Number of athletes: {{ athlete_list|length }}
{% elif athlete_in_locker_room_list %}
    Athletes should be out of the locker room soon!
{% else %}
    No athletes.
{% endif %}

<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% empty %}
    <li>Sorry, no athletes in this list.</li>
{% endfor %}
</ul>

{{ func arg1 arg2}} - call a function
```

URLs

```python
{% url 'some-url-name' v1 v2 %} 
{% url 'myapp:view-name' %}
{% url 'some-url-name' as the_url %}
```

Commenting

```python
{# this won't be rendered #} - Comment

{% comment "Optional note" %} 
    <p>Commented out text with {{ create_date|date:"c" }}</p>
{% endcomment %} - Ignores everything inside,
- Optional note suggests the reason why it was commented
```

# Deploying

[How to deploy Django | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/howto/deployment/)

1. Run **`manage.py check --deploy`**
2. Secret Key:  `SECRET_KEY = os.environ['SECRET_KEY']`
3. Debug:  `DEBUG = os.environ['DEBUG']`
4. Set `ALLOWED_HOSTS`
5. Static files
6. `pip freeze > requirements.txt`

### Secret Key

```python
import os
SECRET_KEY = os.environ['SECRET_KEY']
```

### Debug

```python
import os
DEBUG = os.environ['DEBUG']
```

### Static Files

Static files are automatically served by the development server. In production, you must define a **[STATIC_ROOT](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-STATIC_ROOT)** directory where **[collectstatic](https://docs.djangoproject.com/en/4.0/ref/contrib/staticfiles/#django-admin-collectstatic)** will copy them.  URL to use when referring to static files located in **[STATIC_ROOT](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-STATIC_ROOT)**.  

[How to manage static files (e.g. images, JavaScript, CSS) | Django documentation | Django](https://docs.djangoproject.com/en/4.0/howto/static-files/)

### 

# Extras

### Paginator

Gets, the list of objects of the page number from a larger list

```python
def listing(request):
    contact_list = Contact.objects.all()
    paginator = Paginator(contact_list, 25) # Show 25 contacts per page.

    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    return render(request, 'list.html', {'page_obj': page_obj})
```

### Email

```python
from django.core.mail import send_mail

send_mail(
    'Subject here',
    'Here is the message.',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)
```

### Signals

[Built in Signals - Signals | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/ref/signals/)
Recieving Callback to signal.  Sender is optional.

```python
from django.core.signals import request_finished
from django.dispatch import receiver

@receiver(request_finished, sender=MyModel) # Way 1. to connect to callback.
def my_callback(sender, **kwargs):
    print("Request finished!")
```

Recieving Callback - Method 2. Use either

```python
request_finished.connect(my_callback, dispatch_uid="my_unique_identifier") # Way 2. to connect to callback. USE EITHER
```

Disconnecting Signal

```python
Signal.disconnect()
```

Defining signal

```python
import django.dispatch

pizza_done = django.dispatch.Signal()
```

Sending signal

```python
pizza_done.send(sender=self.__class__, toppings=toppings, size=size)
```

`send_robust` is same as `send`  except robust catches exceptions.  return a list of tuple pairs **`[(receiver, response), ... ]`**

### Logging

[How to configure and use logging | Django documentation | Django (djangoproject.com)](https://docs.djangoproject.com/en/4.0/howto/logging/)

```python
import logging

logger = logging.getLogger(__name__)

def some_view(request):
    ...
    if some_risky_state:
        logger.warning('Platform is running at risk')
```

### Conditional View Processing

Return only those page who have been edited by specifying the last date modified or an ETag(String) in a condition. `condition(etag_func=**None**, last_modified_func=**None**)`

```python
from django.views.decorators.http import condition

def latest_entry(request, blog_id):
    return Entry.objects.filter(blog=blog_id).latest("published").published

@condition(last_modified_func=latest_entry)
def view(request, blog_id):
    ...
```

### Redirects

```python
redirect = Redirect.objects.create(
     site_id=1,
     old_path='/contact-us/',
     new_path='/contact/',
 )
```

### Sessions

Edit the **[MIDDLEWARE](https://docs.djangoproject.com/en/4.0/ref/settings/#std:setting-MIDDLEWARE)** setting and make sure it contains **`'django.contrib.sessions.middleware.SessionMiddleware'`**

`request.session`  - dictionary. 
to check if sessions are working/cookies are enabled in the broswer.

```python
from django.http import HttpResponse
from django.shortcuts import render

def login(request):
    if request.method == 'POST':
        if request.session.test_cookie_worked():
            request.session.delete_test_cookie()
            return HttpResponse("You're logged in.")
        else:
            return HttpResponse("Please enable cookies and try again.")
    request.session.set_test_cookie()
    return render(request, 'foo/login_form.html')
```

## To Add
- User & Authentication
- views
- groups & perms
