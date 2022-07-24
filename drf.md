```python
# Create virtual environment
python -m venv venv-aci-drf

# Install Django and Django REST Framework
pip install django
pip install djangorestframework
pip install django-modeladmin-reorder

# Create Django REST framework project
django-admin startproject aci-drf

# Move into aci-drf folder
cd aci-drf

# Create a Django REST framework application
django-admin startapp aci_app
```

There should now be three folders 
```
aci
|    | manage.py
└───aci
|    | __init__.py
|    | asgi.py
|    | settings.py
|    | urls.py
|    | wsgi.py
└───aci_app
     | __init__.py
     | admin.py
     | apps.py
     | models.py
     | tests.py
     | views.py
```

Rename the first `/aci` folder
```
mv aci aci-drf
```

The file structure should now look like this
```
aci-drf 
|    | manage.py
└───aci 
|    | __init__.py
|    | asgi.py
|    | settings.py
|    | urls.py
|    | wsgi.py
└───aci_app
     | __init__.py
     | admin.py
     | apps.py
     | models.py
     | tests.py
     | views.py
```


Inside of the `/aci-drf`  folder sync the database for the first time
```
python manage.py migrate
```


Then create an initial user. Prompt will request password.
```
python manage.py createsuperuser --email admin@example.com --username admin
```


Open `aci-drf/aci_app/models.py` and add the following code
```python
from django.db import models

class epg(models.Model):
    YES_NO = (
        ('yes', 'yes'),
        ('no', 'no'),
    )

    present_absent = (
        ('present','present'),
        ('absent','absent'),
    )

    flood_proxy = (
        ('flood', 'flood'),
        ('proxy', 'proxy'),
    )

    application_profile = (
        ('{{ fw_routed_ap_name }}', 'FW Routed'),
        ('{{ freely_routed_ap_name }}', 'Freely Routed'),
    )

    contract = (
        ('{{ fw_routed_contract_name }}', 'FW Routed Contract'),
        ('{{ freely_routed_contract_name }}', 'Freely Routed Contract'),
    )

    include = models.CharField(max_length=3, choices=YES_NO)
    state = models.CharField(max_length=7, choices=present_absent)
    vlan_id = models.CharField(max_length=4, unique=True, primary_key=True)
    epg_descr = models.CharField(max_length=50, blank=True)
    bd_descr = models.CharField(max_length=50, blank=True)
    arp_flooding = models.CharField(max_length=3, choices=YES_NO)
    l2_unknown_unicast = models.CharField(max_length=5, choices=flood_proxy)
    unicast_routing = models.CharField(max_length=3, choices=YES_NO)
    ap_name = models.CharField(max_length=40, choices=application_profile)
    preferred_group = models.CharField(max_length=7, choices=YES_NO)


    def __str__(self):
        return self.epg_name

    @property
    def epg_name(self):
        return "ADM-EPG-" + self.vlan_id

    @property
    def bd_name(self):
        return "ADM-BD-" + self.vlan_id
```


Move into the `aci-drf/aci_app` folder and create a new file called `serializers.py`
```
cd aci_app
touch serialisers.py
```


Add this code to the file
```python
from rest_framework import serializers
from aci_app.models import *

class epgSeralizer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = epg
        verbose_name_plural = "EPG"
        fields = [
            'include',
            'state',
            'vlan_id',
            'epg_name',
            'epg_descr',
            'bd_name',
            'bd_descr',
            'arp_flooding',
            'l2_unknown_unicast',
            'unicast_routing',
            'ap_name',
            'preferred_group'
        ]
```

Open `aci-drf/aci_app/views.py`and add the following code
```python
from django.shortcuts import render
from rest_framework import viewsets, permissions
from aci_app.models import *
from aci_app.serializers import * 

class epgViewSet(viewsets.ModelViewSet):
    queryset = epg.objects.all()
    serializer_class = epgSeralizer
    permission_classes = [permissions.IsAuthenticated]
```

Open `aci-drf/aci_app/admin.py` and add this code
```python
from django.contrib import admin
from aci_app.models import epg

admin.site.register(epg)
```

Open `aci-drf/aci/urls.py` and add this code
```python
from django import views
from django.contrib import admin
from django.urls import path, include

from rest_framework import routers
from aci_app import views

router_aci = routers.DefaultRouter(trailing_slash=False)
router_aci.register(r'epg', views.epgViewSet) 

urlpatterns = [
    path('admin/', admin.site.urls),

    # ------ ACI ------
    path('api/aci/v1/', include(router_aci.urls)),
]
```


Open `aci-drf/aci/settings.py` and add this code (The code below is pieced up and goes in different places in the file).
```python
...
INSTALLED_APPS = [
    ...
    'rest_framework',
    'aci_app',
    'admin_reorder',
]

MIDDLEWARE = [
    ...
    'admin_reorder.middleware.ModelAdminReorder',
]

...

ADMIN_REORDER = (
    {
        'app': 'aci_app',
        'label': 'ACI',
        'models': (
            'aci_app.epg',
        )
    },
)
```

Now run this command to test what has been done so far
```
python manage.py runserver
```

Enter `127.0.0.1:8000/admin` into a web browser to see the result.


It should look something like this
![image](img/%231.png)

You can also enter `127.0.0.1:8000/api/aci/v1` in another tab to see the root of the API.
![image](img/%232.png)


