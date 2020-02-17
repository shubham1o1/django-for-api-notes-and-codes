- Common convention : api/v1 {tracking the version of API}
- **ListAPIView** -> Read only endpoint collection, **RetrieveAPIView** -> Read only single endpoint, **ListCreateAPIView** -> read/write endpoint, **RetrieveUpdateDestroyAPIView** -> read, update, delete
- Code in views:
```python
# posts/views.py

# Import generics, models and serializers
from rest_framework import generics
from .models import Post
from .serializers import PostSerializer

class PostList(generics.ListCreateAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer

class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```
- All we have to do is update our generic view to radically change the behavior of a given API endpoint.
- api/v1 => list and create endpoint
- api/v1/1 => list specific, update and delete viewpoint

---

## Perimissions
Django REST Framework ships with several out-of-the-box permissions settings that we can use to secure our API. These can be applied at a project-level, a view-level, or at any individual model level.

### Add log in to the browsable API:
Within the project-level urls.py file, add a new URL route that includes rest_framework.urls. Somewhat confusingly, the actual route specified can be anything we want; what matters is that rest_framework.urls is included somewhere. We will use the route api-auth since that matches official documentation, but we could just as easily use anything-we-want and everything would work just the same.
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    ...
    path('api-auth/', include('rest_framework.urls')),
]
```

### AllowAny:
- Dont expose API ko everyone
- previously we set project level permission to AllowAny in settings.py as:
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```
### View-Level Permissions:
- We want to restrict API to authenticated users. (Possible to do from project-level, view-level, object-level)
- import permission and add permission_classes field to each view
    ```python
    from rest_framework import generics, permissions
    ...

    class PostList(generics.ListCreateAPIView):
        permission_classes = (permissions.IsAuthenticated,)
        ...

    class PostDetail(generics.RetrieveUpdateDestroyAPIView):
        permission_classes = (permissions.IsAuthenticated,)
        ...
    ```
- Here's what browsable API shows at the endpoint:

![Authentication credentials](picture_notes/unauthorized.JPG)

### Project-Level Permissions:
Django REST Framework ships with a number of built-in project-level permissions settings we can use, including:
- **AllowAny** - any user, authenticated or not, has full access
- **IsAuthenticated** - only authenticated, registered users have access
- **IsAdminUser** - only admins/superusers have access
- **IsAuthenticatedOrReadOnly** - unauthorized users can view any page, but only authenticated users have write, edit, or delete privileges 

Implementing any of these four settings requires updating the ```DEFAULT_PERMISSION_CLASSES``` setting and refreshing our web browser
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]
}
```
- You can delete the permission_classes in views

### Custom Permissions