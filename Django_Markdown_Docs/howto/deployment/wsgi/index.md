# How to deploy with WSGI

Django’s primary deployment platform is [WSGI](https://wsgi.readthedocs.io/en/latest/), the Python standard for web
servers and applications.

Django’s [`startproject`](../../../ref/django-admin.md#django-admin-startproject) management command sets up a minimal default
WSGI configuration for you, which you can tweak as needed for your project,
and direct any WSGI-compliant application server to use.

Django includes getting-started documentation for the following WSGI servers:

* [How to use Django with Gunicorn](gunicorn.md)
* [How to use Django with uWSGI](uwsgi.md)
* [How to use Django with Apache and `mod_wsgi`](modwsgi.md)
* [How to authenticate against Django’s user database from Apache](apache-auth.md)

## The `application` object

The key concept of deploying with WSGI is the `application` callable which
the application server uses to communicate with your code. It’s commonly
provided as an object named `application` in a Python module accessible to
the server.

The [`startproject`](../../../ref/django-admin.md#django-admin-startproject) command creates a file
`<project_name>/wsgi.py` that contains such an `application` callable.

It’s used both by Django’s development server and in production WSGI
deployments.

WSGI servers obtain the path to the `application` callable from their
configuration. Django’s built-in server, namely the [`runserver`](../../../ref/django-admin.md#django-admin-runserver)
command, reads it from the [`WSGI_APPLICATION`](../../../ref/settings.md#std-setting-WSGI_APPLICATION) setting. By default, it’s
set to `<project_name>.wsgi.application`, which points to the `application`
callable in `<project_name>/wsgi.py`.

## Configuring the settings module

When the WSGI server loads your application, Django needs to import the
settings module — that’s where your entire application is defined.

Django uses the [`DJANGO_SETTINGS_MODULE`](../../../topics/settings.md#envvar-DJANGO_SETTINGS_MODULE) environment variable to
locate the appropriate settings module. It must contain the dotted path to the
settings module. You can use a different value for development and production;
it all depends on how you organize your settings.

If this variable isn’t set, the default `wsgi.py` sets it to
`mysite.settings`, where `mysite` is the name of your project. That’s how
[`runserver`](../../../ref/django-admin.md#django-admin-runserver) discovers the default settings file by default.

#### NOTE
Since environment variables are process-wide, this doesn’t work when you
run multiple Django sites in the same process. This happens with mod_wsgi.

To avoid this problem, use mod_wsgi’s daemon mode with each site in its
own daemon process, or override the value from the environment by
enforcing `os.environ["DJANGO_SETTINGS_MODULE"] = "mysite.settings"` in
your `wsgi.py`.

## Applying WSGI middleware

To apply [**WSGI middleware**](https://peps.python.org/pep-3333/#middleware-components-that-play-both-sides) you can wrap the application
object. For instance you could add these lines at the bottom of
`wsgi.py`:

```default
from helloworld.wsgi import HelloWorldApplication

application = HelloWorldApplication(application)
```

You could also replace the Django WSGI application with a custom WSGI
application that later delegates to the Django WSGI application, if you want
to combine a Django application with a WSGI application of another framework.
