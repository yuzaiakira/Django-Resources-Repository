## Installation
## create Django project
first create virtual environment :
```console
virtualenv envname
```

then active environment:
```console
envname\scripts\activate
```

install Django
```console
pip install django
```

Finally, create the Django project:
```console
django-admin startproject panme
```
### install app
To create a new app, use the following command:

```console
python manage.py startapp appname
```

Then edit the `settings.py` file in the main project and add the app name to the list below:

```python
#settings.py
INSTALLED_APPS = [
	# ...
	
    'appname.apps.AppConfig',
]
```

## Models in Django
### Create models
To create models, navigate to your app's directory and edit the `models.py` file.

```python
#models.py
from django.db import models
# basic markdown of models
class appModel (models.Model):
	pass
	# Your fields here

```
####  #### Create Fields for the Model
You can define various fields for your model class. Here are some examples:
 
```python
# Basic text field
name = models.CharField(max_length=100)

# Basic integer field
length = models.IntegerField(null=True)

# storing floating-point numbers.
price = models.FloatField()

# for fixed-point decimal numbers. You can specify the maximum number of digits and the number of decimal places.
price = models.DecimalField(max_digits=10, decimal_places=2)

# True/False values
is_active = models.BooleanField(default=True)


# Choices field
START = 1
END = 2
CANCEL = 3
SALES = 4

status_choices = (
    (START, "Start of game"),
    (END, "End of game"),
    (CANCEL, "Game is canceled"),
    (SALES, "Game is selling"),
)

status = models.IntegerField(choices=status_choices)

# When using in templates, the returned value of the variable is the default. 
# To display the text in templates, you can use:
# {{ context_name.get_status_display }}

# You can also use all variables (START, END, CANCEL, SALES) in templates. 
# For example:
# {% if context_name.status == context_name.START %}

# Upload image
pic = models.ImageField(upload_to="testmedia") 
# Add MEDIA_URL="/media/" and MEDIA_ROOT=os.path.join(BASE_DIR, "media") to the `settings.py` file. 
# Then create a `media` folder in the main directory (the same level as manage.py).

#Used for uploading files. It can handle any type of file.
document = models.FileField(upload_to='documents/')

# Finally, add a `testmedia` folder inside the `media` folder.
# Note: You should also add a view for serving media files.

# Date and time field
date = models.DateTimeField(auto_now_add=True, auto_now=True)
# auto_now_add=True Sets the field to the current date and time when the object is created and does not change on updates.

# auto_now=True Updates the field to the current date and time every time the object is saved.

# just date field
birth_date = models.DateField()

# Email addresses. It validates the input to ensure it is a valid email format.
email = models.EmailField()

#For storing URLs. It validates the input to ensure it is a valid URL format.
website = models.URLField()
```
For more information about additional fields, you can refer to the link below:
[Django model field reference](https://docs.djangoproject.com/en/5.1/ref/models/fields/)

#### Connection Models
To create relationships between models, Django provides several model fields:

##### 1. ForeignKey
Establishes a many-to-one relationship, where each instance of the model can be related to one instance of another model.
```python
#models.py
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=250)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    # or better way
    author = models.ForeignKey('Author', on_delete=models.CASCADE)
```

##### 2. ManyToManyField
Creates a many-to-many relationship, allowing each instance of the model to be related to multiple instances of another model and vice versa.
```python
#models.py
class Tag(models.Model):
    name = models.CharField(max_length=100)

class Article(models.Model):
    title = models.CharField(max_length=250)
    tags = models.ManyToManyField('Tag')
```

##### 3. OneToOneField
Defines a one-to-one relationship, where each instance of the model can be related to exactly one instance of another model.
```python
#models.py
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()
```

the `on_delete` parameter is used with relationship fields to specify the behavior when the referenced object is deleted. Here are the common options:
1. **`models.CASCADE`**: When the referenced object is deleted, all related objects will also be deleted.
2. **`models.PROTECT`**: Prevents deletion of the referenced object if there are any related objects. Raises a `ProtectedError`.
3. **`models.SET_NULL`**: Sets the foreign key to `NULL` when the referenced object is deleted (requires the field to be nullable).
4. **`models.SET_DEFAULT`**: Sets the foreign key to its default value when the referenced object is deleted.
5. **`models.RESTRICT`**: Prevents deletion of the referenced object if there are any related objects, similar to `PROTECT`, but does not raise an error if there are no related objects.
For more information about `on_delete` parameter, you can refer to the link below:
[on_delete](https://docs.djangoproject.com/en/4.1/ref/models/fields/#django.db.models.ForeignKey.on_delete)

#### Some methods in models

we have some commonly used methods in Django models that can help you work with your data effectively:
##### 1. `__str__()` Method:
Defines a human-readable string representation of the model instance, which is useful for displaying the object in the Django admin and other interfaces.
```python
#models.py
class appModel (models.Model):
	# custhom fields
	# .....
	# For show models record text in admin 
	def __str__(self):
		# returned name (name is Field in model)
		return self.name
```

##### 2. `save()` Method:
Saves the current instance to the database. You can override this method to add custom behavior before or after saving.
```python
def save(self, *args, **kwargs):
    # Custom logic here
    super().save(*args, **kwargs)
```

##### 3. `delete()` Method:
Deletes the current instance from the database. You can override this method to add custom behavior before or after deletion.
```python
def delete(self, *args, **kwargs):
    # Custom logic here
    super().delete(*args, **kwargs)
```

##### 4.  `get_absolute_url()` Method:
Returns the URL to access a particular instance of the model. This is often used in templates for linking to detail views.
```python
from django.urls import reverse

def get_absolute_url(self):
    return reverse('model_detail', args=[str(self.id)])

```

##### 5.  `clean()` Method:
Used for custom validation logic. This method is called when you call `full_clean()` on the model instance.
```python
def clean(self):
    if self.age < 0:
        raise ValidationError('Age cannot be negative.')

```

##### 2. `update()` Method:
A queryset method that allows you to update multiple instances of a model at once.
```python
MyModel.objects.filter(condition).update(field_name='new_value')
```

### Migrating App Models
To create migrations for your models, use the following command:
```console
python manage.py makemigrations
```

After creating the migration files, run the following command to apply the migrations and create the corresponding fields in the database:

```console
python manage.py migrate
```

**Note**: The `makemigrations` and `migrate` commands apply to all apps in your project, not just a single app.

### Registering Models in the Admin Panel
To display your model fields in the Django admin panel, you need to register the model classes using the `admin.site.register(modelname)` method in the `admin.py` file.
```python
#admin.py
from django.contrib import admin
from appname.models import appModel, app2Model

# Register your models
admin.site.register(appModel)
admin.site.register(app2Model)
  

```
## URLs in Django
To create URLs in Django, navigate to the `urls.py` file in your main app and add your new URLs to the `urlpatterns` list:

```python
# urls.py
from django.contrib import admin
from django.urls import path
from YourApp.views import YourView, your_view  # Ensure proper naming conventions

urlpatterns = [
	# Djando admin url here :)
    path('admin/', admin.site.urls),
    
    # When the client accesses 'site/url/', Django will run the function associated with 'View'
    path('url/', YourView.as_view(), name='view_name'),  # Use .as_view() for class-based views
    path('your-own-new-url/', your_view, name='your_view_name'),  # Added hyphens for URL readability
    
    # URL pattern to capture a username from the URL path.
	# When a user accesses 'user/akira/',
	path('user/<str:username>/', your_view, name='user_profile'),
	
	# URL pattern to capture a parameter of a specified type from the URL path.
	# Replace 'type' with the desired converter (e.g., 'int', 'str', 'slug', 'and ...') and 'parameter_name' with the variable name.
	# For example, 'url/<int:item_id>/' would capture an integer item ID.
	path('url/<type:parameter_name>/', your_view),

]

```

for showing media files during development, we should add following code to the `urls.py` in project:
```python
# urls.py
from django.conf import settings
from django.conf.urls.static import static


# Serve media files only in DEBUG mode
if settings.DEBUG:
 	urlpatterns += static(settings.MEDIA_URL,document_root=settings.MEDIA_ROOT)
 	```

### Include the `urls.py` in the project’s `urls.py`
For example, if we have an app named `blog` with the following URLs:
```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    # Root URL of this app, e.g., site/blog/
    path('', views.index, name='home'),  # 'name' is useful for dynamic URL referencing in templates
    path('tags/', views.tags, name='tags'),  # URL for the tags view
]

```

and for including apps URLs in the project’s `urls.py`, you should use the following code:
```python
# urls.py
from django.contrib import admin
from django.urls import path, include #new

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')) #new
]
```
## Views in Django
To create a new view in Django, you should edit the `views.py` file in your custom app.
```python
# views.py
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.
def yourView(request):
	return HttpResponse("hello world :)")
```
### Get URLs path in View
Assume you want to capture an integer parameter called `item_id` from the URL:
```python
# urls.py
from django.urls import path
from .views import your_view

urlpatterns = [
    # Capture an integer parameter 'item_id' from the URL
    path('url/<int:item_id>/', your_view, name='item_detail'),
]

```
In the context of the URL pattern we defined, the view function will receive the captured parameter from the URL and can use it to perform various actions, such as querying the database, processing data, or rendering a template.
```python
# views.py
from django.http import HttpResponse

def your_view(request, item_id):
    return HttpResponse(f'The item ID is: {item_id}')

```

### models in View
To use models in your views, you need to import the model class in the `views.py` file and utilize various model methods. For example:
```python
from django.shortcuts import render
from django.http import HttpRespose
from .models import appModel

# Create your views here.
def yourView(request):
	# objects.all(), get all record in DB 
	all = appModel.objects.all()
	return HttpRespose(all)
```

here is the not recommend method for showing list of the DB records
```python
from django.shortcuts import render
from django.http import HttpResponse
from .models import appModel

# Create your views here.
def yourView(request):
	# objects.all(), get all record in DB 
	all = appModel.objects.all()
	htmlbody = """
	<html>
		<body>
			<ul>
			{}
			</ul>
		</body>
	</html>
	""".format('\n'.join('<li>{}</li>'.format(list) for list in all))
	return HttpResponse(htmlbody)
```

### Templates for View
To use HTML files in your Django project, you should follow these steps:
- **Create a `templates` Folder**: In your custom app directory, create a folder named `templates`.

- **Create an App-Specific Folder**: Inside the `templates` folder, create another folder with the same name as your app. This helps Django locate your templates easily. The directory structure should look like this:
	```
	project/
		├── customapp/
		│	├── views.py
		│   ├── models.py
	    │   ├── templates/
		│       └── customapp/
		│           └── home.html
		├── app2/
		└── mainapp/
	```

-  **Create Your HTML Template**: Inside the app-specific folder, create your HTML file (e.g., `your_template.html`).

- **Render the Template in Your View**: In your view function, use the `render` function to return the HTML template as a response. Here’s an example: 
	```python
	# views.py
	from django.shortcuts import render
	from .models import appModel
	
	# Create your views here.
	def yourView(request):
		context = {
			"lists" : appModel.objects.all(),
			"count" : appModel.objects.all().count(),
		}
		return render(request,"customapp/home.html",context)
	```

#### Django templates syntax
Django templates are similar to HTML but include additional features that allow for dynamic content rendering. They utilize template tags, which are used for logic and control flow, and these tags start with `{%` and end with `%}`. For example, you can use conditional statements to display different content based on certain conditions. Additionally, template variables are accessed using double curly braces `{{ }}`, allowing you to display data from the context dictionary passed to the template.

When you pass a context dictionary in the `render` function, you can access its items in the template using the corresponding keys. This enables you to create dynamic web pages that respond to user data and application logic. By combining template tags and variables, you can effectively control the content displayed to users, making your web application interactive and user-friendly.
For example:
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Product List</title>
</head>
<body>
    <h1>Available Products {{count}}</h1>

    {% if lists %}
        <ul>
            {% for product in lists %}
                <li>
                    <strong>{{ product.name }}</strong> - ${{ product.price }}
                    {% if product.in_stock %}
                        <span style="color: green;">In Stock</span>
                    {% else %}
                        <span style="color: red;">Out of Stock</span>
                    {% endif %}
                </li>
            {% endfor %}
        </ul>
    {% else %}
        <p>No products available at the moment.</p>
    {% endif %}

    {% if user.is_authenticated %}
        <p>Welcome back, {{ user.username }}!</p>
    {% else %}
        <p>Hello, Guest! Please <a href="/login/">log in</a> to see more features.</p>
    {% endif %}
</body>
</html>
 
```

For more information about `Django tmplate` tag and filter, you can refer to the link below:
[Django templates](https://docs.djangoproject.com/en/5.1/ref/templates/builtins/)

To use an `ImageField` in a Django model, define it as follows:
```python
# models.py
from django.db import models

class AppModel(models.Model):
    pic = models.ImageField(upload_to="testmedia")

```

To display the image in a template, use the following code:
```html
{# Check if 'lists' contains data #}
{% if lists %}
    {% for list in lists %}
        <li>
            {# Get the URL of the 'pic' field #}
            <img src="{{ list.pic.url }}">
        </li>
    {% endfor %}
{% endif %}

```

Some template syntax examples:
```html
{# Use a method from the model class if it returns a value #}
{{ list.methodClass }}
```

##### Blocks in Template
Django supports HTML template inheritance using blocks. For example, consider the `layout.html` file with the following code:
```HTML
{# layout.html #}
{% load static %}
<html>
    <head>
        <title>{% block titlepage %}No title{% endblock %}</title>
        <link rel="stylesheet" type="text/css" href="{% static 'customapp/style.css' %}">
        <link rel="stylesheet" type="text/css" href="{% static 'customapp/Bootstrap.css' %}">
    </head>
    <body>
        <p>Count is: {{ count }}</p>
        <ul>
            {# Define a block for main content #}
            {# syntax: {% block blockname %} #}
            {% block maincontent %}
                {# Default message if no content is provided #}
                This page is empty
            {% endblock %}
        </ul>
        <script src="{% static 'customapp/jquery.js' %}"></script>
        <script src="{% static 'customapp/index.js' %}"></script>
        {# Block for dynamic scripts #}
        {% block script %}{% endblock %}
    </body>
</html>

```

To use the layout, extend `layout.html` in your template like this:
```html
{% extends "appname/layout.html" %}

{# No titlepage block used; 'No title' is set in the layout #}

{% block maincontent %}
    <p>Count is: {{ count }}</p>
    <ul>
        {# Check if 'lists' contains data #}
        {% if lists %}
            {% for list in lists %}
                <li>
                    {# Display data using the __str__ method #}
                    {{ list }}
                </li>
            {% endfor %}
        {% endif %}
    </ul>
{% endblock %}

{# No script block used as there are no custom scripts #}

```

### Static file for templates 
Static folder is for static file like CSS, JS, fonts and ..., for adding static folder you should be doing like Template folder
```
project/
├── customapp/
│   └── static/
│       └── customapp/
│           └── style.css
├── app2/
└── mainapp/
```

You also need to edit `settings.py` to include the following code:
```python
# settings.py
STATIC_URL = '/static/'
STATIC_ROOT=os.path.join(BASE_DIR,"static")
```

To use static files in your HTML template, include the following code:
```HTML
{% load static %}
<html>
	<head>
	<link rel="stylesheet" type="text/css" href="{% static 'customapp/style.css'%}">
	</head>
	<body>
		{# contect #}
	</body>
</html>
```

## Admin panel 
Django provides a powerful admin panel accessible at `siteurl/admin`. This URL is defined in the `urls.py` file of the main app.
To add an admin user, run the following command:
```console
python manage.py createsuperuser
```

You will be prompted to enter a username, email, and password. After creating the superuser, you can log in to the admin panel using your new credentials.

### Transitioning the Admin Panel
To add another language in Django, follow these steps:

1. **Edit `settings.py`:** Update the following variables:
	```python
	# settings.py
	LANGUAGE_CODE = 'fa-ir'
	TIME_ZONE = 'asia/Tehran'
	```

2. **Add `verbose_name` in `apps.py`:** In your app's `apps.py`, add a `verbose_name` attribute to customize the display name:
	```python
	# apps.py
	from django.apps import AppConfig
	
	# the name of the this app is account
	class AccountConfig(AppConfig):
	    default_auto_field = 'django.db.models.BigAutoField'
	    name = 'account'
	    verbose_name = 'Account settings'
```

3. **Add `Meta` class to models in `models.py`:** To provide user-friendly names for your models and their fields in Django, include a `Meta` class in each model within `models.py`. For example, in the `AppModel`, you can define the `Meta` class with `verbose_name` set to "Profile" and `verbose_name_plural` set to "Profiles" to customize how the model appears in the admin panel. Additionally, use the `verbose_name` attribute for each field to specify custom labels,
	```python
	# models.py
	from django.db import models
	
	class AppModel(models.Model):
		class Meta:
			verbose_name = "profile"
			verbose_name_plural = "profiles"
			
		# add verbose_name to the field to dont showing default text
		name =  models.CharField(max_length=100,verbose_name="نام")
		age =   models.IntegerField(null=true)
		phone = models.CharField(max_length=10,null=true)
	```

### Create login view
First, create a template file named `login.html` and add the following code:
```html
<!-- login.html -->
<form action="" method="POST">
    {# Check context to see if login failed #} 
    {% if has_error %}
        <div class="alert alert-danger" role="alert">{{ error }}</div>
    {% endif %}
    
    {# Protect against CSRF attacks #} 
    {% csrf_token %}
    <input class="form-control" name="username" type="text" placeholder="user" />
    <input class="form-control" name="password" type="password" placeholder="pass" />
    <button class="btn btn-primary d-block w-100 mt-3" type="submit" name="submit">Login</button>
</form>

{# Example of using tags #}
{% if user.is_authenticated %}
    ...
{% endif %}

{# URL path in `urls.py` #}
{# NAME OF THE PATH in the `urls.py` #}
{% url 'PostDetails' %}
```

Next, write a simple view in `views.py`:
```python
# views.py
from django.shortcuts import render
from django.contrib.auth import authenticate, login
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse

# simple view, just logined user can see this view
def AccountView(request):
	# check user is login and active?
    if request.user.is_authenticated and request.user.is_active:
        return render(request,"accounts/Account.html")
    else:
        return HttpResponseRedirect(reverse(loginView))

def loginView(request):
	#check method is post?
    if request.method=='POST':
	    # get `post` data, username is name of the  HTML input
        username = request.POST.get('username')
        password = request.POST.get('password')
        #authenticate user 
        user = authenticate(request, username=username, password=password)
        # if authenticate isn't valid(username and password isn't true) return None
        if user is not None:
		    #doing login user
            login(request, user)
            # Redirect client to AccountView 
            return HttpResponseRedirect(reverse(AccountView))
        # if username = password isnt true, so user is None and we cant login the user
        else:
            context = {
               'has_error': True,
                'user': username,
                'error':"nothing",
            }
            return render(request,"accounts/login.html",context)
    # if is method is't post load this render
    else:
        if request.user.is_authenticated and request.user.is_active:
            return HttpResponseRedirect(reverse(AccountView))
        else:
            return render(request,"accounts/login.html",{
               'has_error': False,
            })       
```

### Create Logout View
To create a logout view, add the following code:
```python
# view.py
from django.contrib.auth import logout
from django.urls import reverse

def LogoutView(request):
    logout(request)
    return HttpResponseRedirect(reverse(loginView))
    
```

Finally, add the URLs in `urls.py`:
```python
# urls.py
from django.urls import path
from .views import AccountView, loginView

urlpatterns = [
    path('',AccountView),
    path('login', loginView),
]
```

## some Django command

Here’s a list of useful Django commands that you can use in your projects:
```console
# django help
>python manage.py help

# start a new project
> django-admin startproject projectname

# run project (development server)
> python manage.py runserver 

# create super user
> python manage.py createsuperuser

# make new app
>python manage.py startapp appname

# create new migrations based on changes in models
> python manage.py makemigrations

# apply migrate
> python manage.py migrate

# show the current state of migrations
> python manage.py showmigrations

# check for any issues in your project
> python manage.py check

# open the Django shell
>python manage.py shell

# run tests
> python manage.py test

# collect static files into a single directory
> python manage.py collectstatic

# dump the database to a JSON file
> python manage.py dumpdata > data.json

# load data from a JSON file into the database
> python manage.py loaddata data.json

# generate a new secret key
> python -c "from django.core.management import utils; print(utils.get_random_secret_key())"

```

