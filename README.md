[![Downloads](https://pypip.in/download/django-rest-encrypted-lookup/badge.svg)](https://pypi.python.org/pypi/django-rest-encrypted-lookup/) [![Build Status](https://travis-ci.org/InterSIS/django-rest-encrypted-lookup.svg?branch=master)](https://travis-ci.org/InterSIS/django-rest-encrypted-lookup)
django-rest-encrypted-lookup
=============

django-rest-encrypted-lookup provides replacement ViewSets, Serializers, and Fields that replace your IntegerField pk or id lookups with encrypted string lookups.

The json representation of a poll:
```
    {
        "id": 5,
        "questions": [
                      2,
                      10,
                      12,
        ]
    }
```
becomes:
```
    {
        "id": "4xoh7gja2mtvz3i47ywy5h6ouu",
        "questions": [
                      "t6y4zo26vutj4xclh5hpy3sc5m",
                      "hp7c75ggggiwv6cs5zc4mpzeoe",
                      "rqq5a2evfokyo7tz74loiu3bcq",
        ]
    }
```

The endpoint:

```
    /api/polls/5/
```
becomes:
```
    /api/polls/4xoh7gja2mtvz3i47ywy5h6ouu/
```

Encrypted lookups are *not* appropriate to use as a security measure against users guessing object URIs. Encrypted
lookups *are* appropriate to use as a non-security-critical method of obfuscating object pks/ids.


Installation
===============

Install the module in your Python distribution or virtualenv:

    $ pip install django-rest-encrypted-lookup

Add the application to your `INSTALLED_APPS`:

```
  INSTALLED_APPS = (
  ...
  "rest_framework_encrypted_lookup",
  ...
  )
```

And in settings.py:

```
  LOOKUP_FIELD = 'id'  # String value name of your drf lookup field, generally 'id' or 'pk'
  
  # Choose your own 
  # Protect ID_ENCRYPT_SECRET as closely as your SECRET_KEY
  # DO NOT set ID_ENCRYPT_SECRET to be equal to your SECRET_KEY
  # If ID_ENCRYPT_SECRET changes, then your lookup strings will change
  ID_ENCRYPT_SECRET = b'16Secretchars+++'
```

How it Works
============

Encrypted lookup strings are *not* stored in the database in association with the objects they represent. Encrypted
lookups are generated by the model serializers during response composition. Encrypted lookups presented in the endpoint
URI are decrypted in the call to dispatch, and encrypted lookups presented in data fields are decrypted by the model
deserializers.

Encryption is provided by the PyCrypto AES library.

Use
===

django-rest-encrypted-lookup provides two field classes:

    from rest_framework_encrypted_lookup.fields import EncryptedLookupField, EncryptedLookupRelatedField
    
a new serializer class:

    from rest_framework_encrypted_lookup.serializers import EncryptedLookupModelSerializer
    
and a generic view class:

    from rest_framework_encrypted_lookup.views import EncryptedLookupGenericViewSet
    
Use these in place of the classes provided with Django Rest Framework 3.
    
Example:

``` 
    # models.py
    ...
    class Poll(models.Model):
        ...
        
        
    # serializers.py
    ...
    class PollSerializer(EncryptedLookupModelSerializer):

        class Meta:
            model = Poll
            
            
    # views.py
    ...
    from rest_framework import viewsets
    
    class PollViewSet(EncryptedLookupGenericViewSet,
                      viewsets.mixins.ListModelMixin,
                      viewsets.mixins.RetrieveModelMixin,
                      ...
                     )
        
        queryset = Poll.objects.all()
        serializer_class = PollSerializer
        
        lookup_field = 'id'
```

Of the four classes included in this package, the example above makes use of `EncryptedLookupModelSerializer`, and 
`EncryptedLookupGenericViewSet`.

The fields `EncryptedLookupField`, `EncryptedLookupRelatedField` are used implicitly
by the EncryptedLookupModelSerializer. These fields may also be used explicitly, if needed.

Compatibility
=============

* Django Rest Framework 3
* Django 1.6, 1.7
* Python 2.7, 3.4

Additional Requirements
=======================

* PyCrypto

Todo
====

* Model name salts.
* More flexible secret key format.
* Coverage.
* Wheels.
* Settings dictionary.
* EncryptedLookupHyperlinkedModelSerializer.

Getting Involved
================

Feel free to open pull requests or issues. GitHub is the canonical location of this project.
