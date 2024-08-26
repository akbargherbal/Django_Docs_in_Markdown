# How to use Django with uWSGI

[uWSGI](https://uwsgi-docs.readthedocs.io/) is a fast, self-healing and developer/sysadmin-friendly application
container server coded in pure C.

#### SEE ALSO
The uWSGI docs offer a [tutorial](https://uwsgi.readthedocs.io/en/latest/tutorials/Django_and_nginx.html) covering Django, nginx, and uWSGI (one
possible deployment setup of many). The docs below are focused on how to
integrate Django with uWSGI.

## Prerequisite: uWSGI

The uWSGI wiki describes several [installation procedures](https://uwsgi-docs.readthedocs.io/en/latest/Install.html). Using pip, the
Python package manager, you can install any uWSGI version with a single
command. For example:

```console
# Install current stable version.
$ python -m pip install uwsgi

# Or install LTS (long term support).
$ python -m pip install https://projects.unbit.it/downloads/uwsgi-lts.tar.gz
```

### uWSGI model

uWSGI operates on a client-server model. Your web server (e.g., nginx, Apache)
communicates with a `django-uwsgi` “worker” process to serve dynamic content.

### Configuring and starting the uWSGI server for Django

uWSGI supports multiple ways to configure the process. See uWSGI’s
[configuration documentation](https://uwsgi.readthedocs.io/en/latest/Configuration.html).

Here’s an example command to start a uWSGI server:

```shell
uwsgi --chdir=/path/to/your/project \
    --module=mysite.wsgi:application \
    --env DJANGO_SETTINGS_MODULE=mysite.settings \
    --master --pidfile=/tmp/project-master.pid \
    --socket=127.0.0.1:49152 \      # can also be a file
    --processes=5 \                 # number of worker processes
    --uid=1000 --gid=2000 \         # if root, uwsgi can drop privileges
    --harakiri=20 \                 # respawn processes taking more than 20 seconds
    --max-requests=5000 \           # respawn processes after serving 5000 requests
    --vacuum \                      # clear environment on exit
    --home=/path/to/virtual/env \   # optional path to a virtual environment
    --daemonize=/var/log/uwsgi/yourproject.log      # background the process
```

This assumes you have a top-level project package named `mysite`, and
within it a module `mysite/wsgi.py` that contains a WSGI `application`
object. This is the layout you’ll have if you ran `django-admin
startproject mysite` (using your own project name in place of `mysite`) with
a recent version of Django. If this file doesn’t exist, you’ll need to create
it. See the [How to deploy with WSGI](index.md) documentation for the default
contents you should put in this file and what else you can add to it.

The Django-specific options here are:

* `chdir`: The path to the directory that needs to be on Python’s import
  path – i.e., the directory containing the `mysite` package.
* `module`: The WSGI module to use – probably the `mysite.wsgi` module
  that [`startproject`](../../../ref/django-admin.md#django-admin-startproject) creates.
* `env`: Should probably contain at least [`DJANGO_SETTINGS_MODULE`](../../../topics/settings.md#envvar-DJANGO_SETTINGS_MODULE).
* `home`: Optional path to your project virtual environment.

Example ini configuration file:

```ini
[uwsgi]
chdir=/path/to/your/project
module=mysite.wsgi:application
master=True
pidfile=/tmp/project-master.pid
vacuum=True
max-requests=5000
daemonize=/var/log/uwsgi/yourproject.log
```

Example ini configuration file usage:

```shell
uwsgi --ini uwsgi.ini
```

See the uWSGI docs on [managing the uWSGI process](https://uwsgi-docs.readthedocs.io/en/latest/Management.html) for information on
starting, stopping and reloading the uWSGI workers.
