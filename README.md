# Django REST Framework - Build an API from Scratch

From this [tutorial](https://www.youtube.com/watch?v=i5JykvxUk_A&list=PL_c9BZzLwBRLCpTc20e2pFT1lmbnevegR&index=2)

## VENV

## Dependencies

```shell
pip install django
pip install djangorestframework
```

## Start a project

```shell
django-admin startproject <name> .
python manage.py runserver
python manage.py migrate
python manage.py createsuperuser
```

## Add a model

```py
from django.db import models

class Drink(models.Model):
    name = models.CharField(max_length=200)
    description = models.CharField(max_length=500)
```

in settings.py add the name of the app to INSTALLED_APPS

```shell
python manage.py makemigrations <name>
```

apply to db

```shell
python manage.py migrate
```

## admin.py

```py
from django.contrib import admin
from .models import Drink

admin.site.register(Drink)
```

re-run server

## Add djangorestframework to installed apps in settings.py

```py
INSTALLED_APPS = [
    "rest_framework",
    # ...
]
```

## Serializers.py

```py
from rest_framework import serializers
from .models import Drink


class DrinkSerializer(serializers.ModelSerializer):
    class Meta:
        model = Drink
        fields = ['id', 'name', 'description']

```

## End points

Add views.py

```py
from .models import Drink
from .serializers import DrinkSerializer
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status


@api_view(['GET', 'POST'])
def drinks(request, format=None):
    if request.method == 'GET':
        drinks = Drink.objects.all()
        serializer = DrinkSerializer(drinks, many=True)
        return Response({'drinks': serializer.data}, status=status.HTTP_200_OK)
    elif request.method == 'POST':
        serializer = DrinkSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    return Response(status=status.HTTP_405_METHOD_NOT_ALLOWED)

@api_view(['GET', 'PUT', 'DELETE'])
def drink(request, pk, format=None):
    try:
        drink = Drink.objects.get(pk=pk)
    except Drink.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)
    if request.method == 'GET':
        serializer = DrinkSerializer(drink)
        return Response(serializer.data)
    elif request.method == 'PUT':
        serializer = DrinkSerializer(drink, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        drink.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
    return Response(status=status.HTTP_405_METHOD_NOT_ALLOWED)
```

In urls.py add paths

```py
from django.contrib import admin
from django.urls import path
from drinks import views
from rest_framework.urlpatterns import format_suffix_patterns

urlpatterns = [
    path("admin/", admin.site.urls),
    path("drinks/", views.drinks),
    path("drinks/<int:pk>", views.drink)
]

urlpatterns = format_suffix_patterns(urlpatterns) # allows to see json format in browser, e.g. "drinks/1.json/"
```
