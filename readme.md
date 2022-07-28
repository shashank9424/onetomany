
## Models.py

```python
from django.db import models

from django.db import models

# Create your models here.

class base(models.Model):
    name=models.CharField(max_length=550)
    record=models.ForeignKey("users",on_delete=models.CASCADE,blank=True,null=True,related_name='user_hobby',)

class users(models.Model):
    username=models.CharField(max_length=550)
    email=models.EmailField()
```
## Serializers.py

```python
from rest_framework import serializers
from .models import *

class baseSerializer(serializers.ModelSerializer):
    class Meta:
        model=base
        fields=("id","name","record")

class usersSerializer(serializers.ModelSerializer):
    RELATED_NAME="user_hobby"
    BASE_MODEL="users"
    RELATED_MODEL="base"
    user_hobby= baseSerializer(many=True)
    class Meta:
        model=users
        fields="__all__" 

    def create(self, validated_data):
        user_hobby = validated_data.pop('user_hobby')
        record = users.objects.create(**validated_data)
        
        for account in user_hobby:
            base.objects.create(record=record,**account)
        return record
    def update(self, instance, validated_data):
        user_hobby=validated_data.pop("user_hobby")
        print(user_hobby)
        instance.username = validated_data.get('username', instance.username)
        instance.email = validated_data.get('email', instance.email)
        instance.otp = validated_data.get('otp', instance.otp)
        instance.activation_key = validated_data.get('activation_key', instance.activation_key)
        instance.save()

        user_hobby_with_same_instance=base.objects.filter(record=instance.pk).values_list('id', flat=True)
        hobbies_id_pool = []
        temp=0
        for hobby in user_hobby:
            
            if user_hobby_with_same_instance:
                
                inst=user_hobby_with_same_instance
                try:
                    print(temp)
                    base.objects.filter(id=inst[temp]).exists()
                    hobby_instance = base.objects.get(id=inst[temp])
                    hobby_instance.name = hobby.get('name', hobby_instance.name)
                    hobby_instance.save()
                    hobbies_id_pool.append(hobby_instance.id)
                except:
                    hobbies_instance = base.objects.create(record=instance, **hobby)
                    hobbies_id_pool.append(hobbies_instance.id)
            else:
                hobbies_instance = base.objects.create(record=instance, **hobby)
                hobbies_id_pool.append(hobbies_instance.id)
            temp += 1
        for hobby_id in user_hobby_with_same_instance:
            if hobby_id not in hobbies_id_pool:
                base.objects.filter(pk=hobby_id).delete() 
        return instance
                =models.ForeignKey(baseModel,on_delete=models.CASCADE,blank=True,null=True,related_name='skill',)
    
```

