# Todo APP 

- Create project, app and then migrate. Register app in settings.py
- Create models for todo(title and body), makemigrations and migrate, 
- Create superuser, register app with admin, add item from admin panel
- Install DRF, register DRF in INSTALLED_APPS

- #### Configuring DRF :
    - In the bottom of settings.py, add:

    ```python
        REST_FRAMEWORK = {
            'DEFAULT_PERMISSION_CLASSES': [
            'rest_framework.permissions.AllowAny',
            ]
        }
    ```

    - We configure DRF under REST_FRAMEWORK, here we are explicitly setting permissions to **AllowAny**
    - Numerous default settigs are present which are implicitly applied, but they are not apt for production.

- Create URL, View and serializer
    - URL (project level):

        ```python
            urlpatterns = [
                path('api/', include('todos.urls')),
            ]
        ```
    - URL (app's):

        ```python
            from .views import ListTodo, DetailTodo

            urlpatterns = [
                path('<int:pk>/', DetailTodo.as_view()),
                path('', ListTodo.as_view()),
            ]
        ```
    - **SERIALIZERS**:
        - to transform our data from the models into JSON that will be outputted at the URLs
        - Django REST Framework ships with a powerful built-in serializers class that we can quickly extend
        - Create serializers.py inside todos app, update with following code:
        ```python
        # Import seriliazers from DRF
        from rest_framework import serializers
        # Import model classes
        from .models import Todo


        class TodoSerializer(serializers.ModelSerializer):
            class Meta:
                model = Todo # Specify which model to use 
                fields = ('id', 'title', 'body',) # Specify fields
        ```
        - **Meta Class**: The Meta class of Models and ModelForms can be thought of as something similar to a class or function decorator. The Meta class is referenced during the construction of the form/object instance before the class definition itself. You could try to do the same work by overriding the class __init__ function, but 1) that would be a lot of extra work to replicate something already being done for you by the Meta class, and 2) any such implementation is unlikely to be as efficient or clean as the Meta class. (source: [link](https://www.quora.com/Why-do-we-use-the-class-Meta-inside-the-ModelForm-in-Django))

    - Views: 
        - Django REST Framework ships with generic views for common use cases
        - Update todos/views.py code:
        ```python
        # Import DRF's generic views
        from rest_framework import generics

        from .models import Todo
        from .serializers import TodoSerializer

        class ListTodo(generics.ListAPIView):
        # Displays all model instances
            queryset = Todo.objects.all()
            serializer_class = TodoSerializer
        
        class DetailTodo(generics.RetrieveAPIView):
        # Display single model instance
            queryset = Todo.objects.all()
            serializer_class = TodoSerializer
        ```
- Browsable API : Ships with DRF, go to localhost:8000/api and browse all todo or /api/1 for specific todo

### CORS :
- Cross-Origin Resource Sharing (CORS)
- Whenever a client interacts with an API hosted on a different domain (mysite.com vs yoursite.com) or port (localhost:3000 vs localhost:8000) there are potential security issues.
- Specifically, CORS requires the server to include specific HTTP headers that allow for the client to determine if and when cross-domain requests should be allowed.
- use middleware (django-cors-headers) that will automatically include the appropriate HTTP headers based on our settings.
- Install using: ```pip install django-cors-headers``` , register in INSTALLED_APPS, add in MIDDLEWARE and create CORS_ORIGIN_WHITELIST
```python
# todo_project/settings.py
INSTALLED_APPS = [
    ...
    'corsheaders',
    ...
]

MIDDLEWARE = [
    ...
    'corsheaders.middleware.CorsMiddleware',
    ...
]
CORS_ORIGIN_WHITELIST = (
    'http://localhost:3000', # React default port
    'http://localhost:8000', # Django DP
)
```
- It’s very important that ```corsheaders.middleware.CorsMiddleware``` appears in the proper location. That is above ```django.middleware.common.CommonMiddleware``` in the MIDDLEWARE setting since middlewares are loaded top-to-bottom


### Tests
Code:
```python
# todos/tests.py
from django.test import TestCase
from .models import Todo

class TodoModelTest(TestCase):
    @classmethod
    def setUpTestData(cls):
        Todo.objects.create(title='first todo', body='a body here')

    def test_title_content(self):
        todo = Todo.objects.get(id=1)
        expected_object_name = f'{todo.title}'
        self.assertEquals(expected_object_name, 'first todo')

    def test_body_content(self):
        todo = Todo.objects.get(id=1)
        expected_object_name = f'{todo.body}'
        self.assertEquals(expected_object_name, 'a body here')
```
- uses Django’s built-in TestCase class. First we set up our data in setUpTestData and then write two new tests. Then run the tests with the ```python manage.py test``` command.

