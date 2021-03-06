============================
Start building your own shop
============================

For simplicity, let's assume you're building a new e-commerce project from
scratch and have decided to use Oscar.  Let's call this shop 'frobshop'

.. tip::

    You can always review the set-up of the `Sandbox site`_ in case you have
    trouble with the below instructions.

.. _`Sandbox site`: https://github.com/tangentlabs/django-oscar/tree/releases/0.2/sandbox

Install by hand
===============

Install Oscar (which will install Django as a dependency), then create the
project::

    pip install django-oscar
    django-admin.py startproject frobshop

This will create a folder ``frobshop`` for your project.

Settings
--------

Now edit your settings file ``frobshop.frobshop.settings.py`` to specify a
database (we use SQLite for simplicity)::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': 'db.sqlite3',
            'USER': '',
            'PASSWORD': '',
            'HOST': '',
            'PORT': '',
        }
    }

then add ``oscar.apps.basket.middleware.BasketMiddleware`` to ``MIDDLEWARE_CLASSES``, and
set ``TEMPLATE_CONTEXT_PROCESSORS`` to::

    TEMPLATE_CONTEXT_PROCESSORS = (
        "django.contrib.auth.context_processors.auth",
        "django.core.context_processors.request",
        "django.core.context_processors.debug",
        "django.core.context_processors.i18n",
        "django.core.context_processors.media",
        "django.core.context_processors.static",
        "django.contrib.messages.context_processors.messages",
        'oscar.apps.search.context_processors.search_form',
        'oscar.apps.promotions.context_processors.promotions',
        'oscar.apps.checkout.context_processors.checkout',
        'oscar.core.context_processors.metadata',
    ) 

Next, modify ``INSTALLED_APPS`` to be a list, add ``South`` and append Oscar's core apps::

    from oscar import get_core_apps
    INSTALLED_APPS = [
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.sites',
        'django.contrib.messages',
        'django.contrib.flatpages',
        ...
        'south',
    ] + get_core_apps()

Note that Oscar requires ``django.contrib.messages`` and
``django.contrib.flatpages`` which aren't included by default.

Now set your auth backends to::

    AUTHENTICATION_BACKENDS = (
        'oscar.apps.customer.auth_backends.Emailbackend',
        'django.contrib.auth.backends.ModelBackend',
    )

to allow customers to sign in using an email address rather than a username.

Modify your ``TEMPLATE_DIRS`` to include the main Oscar template directory::

    from oscar import OSCAR_MAIN_TEMPLATE_DIR
    TEMPLATE_DIRS = TEMPLATE_DIRS + (OSCAR_MAIN_TEMPLATE_DIR,) 

Oscar currently uses Haystack for search so you need to specify::

    HAYSTACK_CONNECTIONS = {
        'default': {
            'ENGINE': 'haystack.backends.simple_backend.SimpleEngine',
        },
    }

When moving towards production, you'll obviously need to switch to a real search
backend.

The last addition to the settings file is to import all of Oscar's default settings::

    from oscar.defaults import *

URLs
----

Alter your ``frobshop/urls.py`` to include Oscar's URLs::

    from django.conf.urls import patterns, include, url
    from oscar.app import shop

    urlpatterns = patterns('',
        (r'', include(shop.urls))
    )

Database
--------

Then create the database and the shop should be browsable::

    python manage.py syncdb --noinput
    python manage.py migrate

You should now have a running Oscar install that you can browse.


Install using Tangent's boilerplate django project
==================================================

The easiest way to get started is to use Tangent's `template django project`_
although it is tailored to an agency structure which may not suit everyone.

.. _`template django project`: https://github.com/tangentlabs/tangent-django-boilerplate

Set up a virtualenv, and create a new project using the ``startproject``
management command::

    mkvirtualenv frobshop
    pip install Django
    django-admin.py startproject frobshop \
        --template=https://github.com/tangentlabs/tangent-django-boilerplate/zipball/master 

This will create a folder ``frobshop`` which is an entire templated project that
follows Tangent's conventions.  The structure is::

    frobshop/
        docs/
        www/
            conf/
            deploy/
            public/
            static/
            templates/
            manage.py
            settings.py
            settings_test.py
            urls.py
            urls_oscar.py
        README.rst
        fabconfig.py
        fabfile.py
        deploy-to-test.sh
        deploy-to-stage.sh
        deploy-to-prod.sh

Replace a few files with Oscar-specific versions (the templated project can be
used for non-Oscar projects too)::

    mv frobshop/www/urls{_oscar,}.py
    mv frobshop/www/deploy/requirements{_oscar,}.txt
    mv frobshop/www/conf/default{_oscar,}.py

Install dependencies::

    cd frobshop/www
    pip install -r deploy/requirements.txt

Create database::

    python manage.py syncdb --noinput
    python manage.py migrate

And that should be it.

Next steps
==========

The next step is to implement the business logic of your domain on top of
Oscar.  The fun part.
