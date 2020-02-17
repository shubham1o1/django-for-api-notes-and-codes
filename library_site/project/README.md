# Relevant Notes

every URL like https://python.org/about/ has three potential parts:
- a scheme - https (It tells the web browser how to access resources at the location. For a website this is typically http or https, but it could also be ftp for files, smtp for email, and so)
- a hostname - www.python.org
- and an (optional) path - /about/

### HTTP Verbs:

|  CRUD  | HTTP Verbs | 
|--------|:----------:|
| Create |    Post    |
|  Read  |    Get     |
| Update |    Put     |
| Delete |   Delete   |

### HTTP messages:
Every HTTP message, whether a request or response, has the following format:

```
Response/request line
Headers...
(optional) Body
```
By using HTTP headers we can set various levels of authentication and permission too

Most web pages contain multiple resources that require multiple HTTP request/response cycles. If a webpage had HTML, one CSS file, and an image, three separate trips back-and-forth between the client and server would be required before the complete web page could be rendered in the browser.

### Status Codes:

- 2xx Success - the action requested by the client was received, understood, and accepted
- 3xx Redirection - the requested URL has moved
- 4xx Client Error - there was an error, typically a bad URL request by **the client**
- 5xx Server Error - **the server** failed to resolve a request

### Statelessness:

- HTTP is that it is a stateless protocol
- each request/response pair is completely independent of the previous one.
    #### Pros 
    - No worries about signal loss during req/resp cycle.
    #### Cons
    - State is important (eg: logging in, shoppin cart info is to be stores in states)
- HTTP sends reliable info but is bad at remembering outside req/resp.

### REST:

- REpresentational State Transfer
- is stateless, like HTTP
- supports common HTTP verbs (GET, POST, PUT, DELETE, etc.)
- returns data in either the JSON or XML format

---

# Library Project:
- Create : library project, books app
- Register app in settings.py, migrate
- create book's model, createsuperuser, register to admin site, make migrations, migrate
- Add some books from admin

### Django Rest Framework:

- Installing package: ``` pip install djangorestframework```
- Register in INSTALLED_APPS of settings.py : add ``` 'rest_framework' ```
- Endpoint of API requires **an URL**, **a View** and **a Serializer**
- Create an api app for handling all apis : ```python manage.py startapp api```
- Register to INSTALLED_APPS
- Route project level URL to this path : ```path('api/', include('api.urls')),```
- Create **urls.py** in this app
- Add route to endpoint: ```path('', BookAPIView.as_view()),```
- in views(some developer rename them as apiviews to avoid confusion), do the following

    ```python
    # api/views.py

    # Django REST Framework’s generics class of views
    from rest_framework import generics


    from books.models import Book # models from our books app

    from .serializers import BookSerializer # serializers from our api app

    # create a BookAPIView that uses ListAPIView to create a read-only endpoint
    class BookAPIView(generics.ListAPIView):
        queryset = Book.objects.all() # specify the queryset
        serializer_class = BookSerializer # specify the serializer_class
    ```
#### Serializers:

- translates data into a format that is easy to consume over the internet, typically JSON, and is displayed at an API endpoint. (onvert Django models to JSON)
- make **serializer.py** file within our api app

    ```python
    # api/serializers.py

    # import Django REST Framework’s serializers class
    from rest_framework import serializers

    from books.models import Book

    # extend Django REST Framework’s ModelSerializer into a BookSerializer class
    class BookSerializer(serializers.ModelSerializer):

        class Meta:
            # specifies our database model Book and the database fields
            # we wish to expose: title, subtitle, author, and isbn.
            model = Book
            fields = ('title', 'subtitle', 'author', 'isbn')
    ```

#### Browsable API:
- go to the endpoint, **localhost:8000/api** on a browser where you can see the visualization of the API made by DRF