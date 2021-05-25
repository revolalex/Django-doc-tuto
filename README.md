# Django-doc-tuto

## Table of contents
* [Start a new Django project](#start-a-new-django-project)
* [Project config](#project-config)
* [Create data model](#create-data-model)
* [Admin Pannel](#admin-pannel)
* [Routing](#routing)
* [Function Based Views](#function-based-views)
* [Class Based Views](#class-based-views)


# Cheat Sheet
## Start a new Django project

```py
# Create et access project folder
~$  mkdir project_name
~$  cd project_name

# Create Python virtual env 
~$  python3 -m venv venv

# Activate virtual env
~$  source venv/bin/activate

# Deactivate virtual env
~$  deactivate

# If requirements already exist
~$  pip install -r requirements.txt

# Freeze requirements in a file
~$  pip freeze > requirements.txt

# Install django
~$  pip install django~=3.1.0 

# New django project (from project_name folder)
~$  django-admin startproject config .

# Create app (from project_name folder)
~$  python manage.py startapp app_name

# Migration
~$  python manage.py makemigrations 
~$  python manage.py migrate

# Create superuser
~$  python manage.py createsuperuser

# Start server
~$  python manage.py runserver  => ex.  http://127.0.0.1:8000

# Current project shell example
~$ python manage.py shell
 >>> from app_name.models import User
 >>> user1 = User.objects.first()

# Prepare static folders fpor production
$ python manage.py collectstatic

# Take all data from app blog and export in json
python manage.py dumpdata blog >myapp.json

# Take all data in json file and import in app data table
python manage.py loaddata myapp.json
```

## Project config

```py
# Add app to settings.py
INSTALLED_APPS = [ … , 'app_name' ]

# App templates folder
create folder appfolder/templates/appname

# Project templates folder: 
create folder projectname/templates

# settings.py template config
Project templates settings.py: 
    TEMPLATES = [
        { …
                'DIRS': [BASE_DIR / 'templates', ],
        … }

# Create Static folder: 
project_name\static\

# Static folder (settings.py): 
STATIC_URL = '/static/'
STATICFILES_DIRS = [ BASE_DIR / 'static' ] 
STATIC_ROOT = 'static_root'

# To use PostgresSQL
# pip install psycopg2
# settings.py
DATABASE = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'blog',
        'USER': 'admin',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': '5432'
```

## Create data model

```py
# models.py 
from django.db import models

class Customer(models.Model):

    # database, display
    TYPE_CHOICES = (
        ('Customer', 'Customer'),
        ('Supplier', 'Supplier'),
        ('Student', 'Student'),
    )

    name = models.Charfield('Article name', max_length=120)
    note = models.TextField(blank=True, null = True)
    email = models.EmailField(max_length=255, blank=True, null=True)
    credit = models.FloatField(blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    type = models.CharField(choices=TYPE_CHOICES)

    def __str__(self): 
        return self.name

# User for Auth and more
# Add to settings.py: AUTH_USER_MODEL = 'app_name.User'
from django.contrib.auth.models import AbstractUser
class User(AbstractUser):
    # Inherits AbstractUser fields
    pass

# One-to-Many: (use double quotes if the entity is not yet declare) ex. "Supplier"
supplier = models.ForeignKey(Supplier, blank=True, null=True, on_delete=models.CASCADE)

# Many-to-Many: 
tags = models.ManyToManyField(Tag, blank=True)

# One to One 
User = models.OneToOneField(User, on_delete=models.CASCADE)

# Overwrite save method
def save(self, (*args, **kwargs):
    if not self.slug:
        self.slug = slugify(self.title)

    super().save(*args, **kwargs)
```

## Admin Pannel

```py
# admin.py
from .models import Blog
admin.site.register(Blog)

# Custom model Admin (admin.py): 
class BlogAdmin(admin.ModelAdmin)
    list_display = ("title", "description")
    ordering("date_created",)
    search_fields("title", "description")

# Register app
admin.site.register(Blog, BlogAdmin)
```

## Routing
```py
# APP URLS 
# create urls.py in app_name folder
from django.urls import path
from . import views
from django.conf import settings
from django.conf.urls.static import static

url patterns = [ 
    path('posts', views.index, name='posts.index'),
    path('posts/create/', views.create, name='posts.create',
    path('posts/<int:id>/', views.show, name='posts.show'),
    path('posts/<int:id>/edit/', views.edit, name='posts.edit'),
    path('posts/<int:id>/delete/', views.delete, name='posts.delete'),
] 

# URL Route ex. names conventions
posts.index | posts.create | posts.edit | posts.delete | posts.show

# PROJECT URLS 
# Include the app urls into the project urls (project_name/urls.py) 
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_name.urls'))
]

# Production Static route
urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

## Function Based Views

```py
# views.py
from django.shortcuts import render, redirect
from .models import Post
from .forms import PostForm

def index(request):
    # Get all Posts
    posts = Post.objects.all()

    # Render app template
    return render(request, 'appfolder/index.html', {'posts': posts})

def show(request, id):
    post = Post.objects.get(id=id)
    return render(request, 'appfolder/show.html', {'post': post})

def create(request):
    form = PostForm(request.POST or None)
    if form.is_valid():
        # optionally we can access form data with form.cleaned_data['first_name']
        form.save()
        return redirect('/posts')

    return render(request, 'appfolder/create.html', {'form': form)

def edit(request, id):
    post = Post.objects.get(id=id)
    form = PostForm(request.POST or None, instance=post)
    if form.is_valid():
        form.save()
        return redirect('/posts')

    return render(request, 'appfolder/edit.html', {'form': form)

def delete(request, id):
    post = Post.objects.get(id=id)
    post.delete()
    return redirect('/posts')
````

## Class Based Views

```py
from django.views.generic import TemplateView, ListView, DetailView, CreateView, UpdateView, DeleteView

class LandingPageView(TemplateView):
    template_name = 'landing.html'

    # change context data dict
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Landing Page'
        return context

class PostsListView(ListView):
    model = Post 
    # or 
    queryset = Post.objects.all() 

    # var name object_list (or post_list) will be available in template
    # context_object_name = "posts" to change object list var name

    # by default the view will open post/post_list.html but that can be change
    template_name = 'posts/post_list.html'

class PostsDetailView(DetailView):
    model = Post # object var in template

    # by default the view will try to open post/post_detail.html 
    template_name = 'posts/post_detail.html'


class PostsCreateView(CreateView):
    form_class = PostForm
    template_name = 'posts/post_create.html'

    # default return is detail view
    def get_success_url(self):
        return reverse('posts-list')

    # we can overwrite form data
    def form_valid(self, form):
        if self.request.user.is_authenticated:
            from.instance.author = self.request.user

        return super().form_valid(form)

class PostsUpdateView(UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'posts/post_update.html'
    def get_success_url(self):
        return reverse('post-list')

    # change context data dict
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['submit_text'] = 'Update'
        return context


class PostsDeleteView(DeleteView):
    model = Post
    template_name = 'posts/post_delete.html'
    success_url = reverse_lazy('posts-list')

# Urls.py route declaration
path('<int:pk>/update/', PostsUpdateView.as_view(), name='post-update')
```

### Virtual environment 
```py
$ python3 -m venv venv
$ source venv/bin/activate
```

### Installer Django
```py
$ pip install django
```

### Créer un project
```py
$ django-admin startproject mysite
```

### Lancer le serveur
```py
$ python manage.py runserver
```

### Créer une application
```py
$ python manage.py startapp myapp
```

https://docs.djangoproject.com/fr/3.2/intro/tutorial01/


