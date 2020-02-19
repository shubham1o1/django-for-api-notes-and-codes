# Viewsets and Routers:

- Abstraction, replace related Views, auto generate url and Speed up API dev. 
- many endpoint less code. 
- Our endpoints:

| Endpoints                         | HTTP Verb |
|-----------------------------------|-----------|
| /                                 | GET       |
| /:pk                              | GET       |
| /rest-auth/registration           | POST      |
| /rest-auth/login                  | POST      |
| /rest-auth/logout                 | GET       |
| /rest-auth/password/reset         | POST      |
| /rest-auth/password/reset/confirm | POST      |

- Add two more to list all users and individual users. 
- serializers.py:
```python
from django.contrib.auth import get_user_model

...
class UserSerializer(serializers.ModelSerializer): 
    
    class Meta:
        model = get_user_model()
        fields = ('id', 'username',)
```
- views.py:
```python
from django.contrib.auth import get_user_model
...
from .serializers import PostSerializer, UserSerializer
...
class PostList(generics.ListCreateAPIView):
    permission_classes = (permissions.IsAuthenticated,)
    queryset = Post.objects.all()
    serializer_class = PostSerializer

class PostDetail(generics.RetrieveUpdateDestroyAPIView):
    permission_classes = (permissions.IsAuthenticated,)
    permission_classes = (IsAuthorOrReadOnly,)
    queryset = Post.objects.all()
    serializer_class = PostSerializer

class UserList(generics.ListCreateAPIView): 
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer

class UserDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer
```
- There is quite a bit of repetition here. Both Post views and User views have the exact same ```queryset``` and ```serializer_class```. Maybe those could be combined in some way to save code. 
- urls.py:
```python
urlpatterns = [
    path('users/', UserList.as_view()),
    path('users/<int:pk>/', UserDetail.as_view()),
]
```
- Our user list endpoint is located at http://127.0.0.1:8000/api/v1/users/
- A user detail endpoint is available at the primary key for each user. So our superuser account is located at: http://127.0.0.1:8000/api/v1/users/1/.

### Viewsets:
- Combine the logic for multiple related views into a single class. In other words, on viewset can replace multiple views. We lose the readability.
- Updated views.py:
```python
from rest_framework import viewsets

class PostViewSet(viewsets.ModelViewSet):
    permission_classes = (IsAuthorOrReadOnly,)
    queryset = Post.objects.all()
    serializer_class = PostSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = get_user_model().objects.all()
    serializer_class = UserSerializer
```
- At the top instead of importing generics from rest_framework we are now importing viewsets on the second line. Then we are using ModelViewSet which provides both a list view and a detail view for us. And we no longer have to repeat the same queryset and serializer_class for each view as we did previously!

### Routers:
- Routers work directly with viewsets to automatically generate URL patterns for us.
- Our current posts/urls.py file has four URL patterns: two for blog posts and two for users. We can instead adopt a single route for each viewset. So two routes instead of four URL patterns.
- Django REST Framework has two default routers: SimpleRouter and DefaultRouter. We will use SimpleRouter but it’s also possible to create custom routers for more advanced functionality.
- urls.py:
```python
from django.urls import path
from rest_framework.routers import SimpleRouter

from .views import UserViewSet, PostViewSet

router = SimpleRouter()
router.register('users', UserViewSet, basename='users')
router.register('', PostViewSet, basename='posts')

urlpatterns = router.urls
```
- On the top line SimpleRouter is imported, along with our views. The router is set to SimpleRouter and we "register" each viewset for Users and Posts. Finally we set our URLs to use the new router. 

- Ultimately the decision of when to add viewsets and routers to your project is quite subjective. A good rule of thumb is to start with views and URLs. As your API grows in complexity if you find yourself repeating the same endpoint patterns over and over again, then look to viewsets and routers. Until then, keep things simple.

# Schemas and Documentation