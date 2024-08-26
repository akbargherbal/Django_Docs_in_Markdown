# Writing your first Django app, part 8

This tutorial begins where [Tutorial 7](tutorial07.md) left off. We’ve
built our web-poll application and will now look at third-party packages. One of
Django’s strengths is the rich ecosystem of third-party packages. They’re
community developed packages that can be used to quickly improve the feature set
of an application.

This tutorial will show how to add [Django Debug Toolbar](https://pypi.org/project/django-debug-toolbar/), a commonly used third-party package. The Django Debug
Toolbar has ranked in the top three most used third-party packages in the
Django Developers Survey in recent years.

## Installing Django Debug Toolbar

Django Debug Toolbar is a useful tool for debugging Django web applications.
It’s a third-party package maintained by the [Jazzband](https://jazzband.co) organization. The toolbar helps you understand how your
application functions and to identify problems. It does so by providing panels
that provide debug information about the current request and response.

To install a third-party application like the toolbar, you need to install
the package by running the below command within an activated virtual
environment. This is similar to our earlier step to [install Django](../topics/install.md#installing-official-release).

```console
$ python -m pip install django-debug-toolbar
```

Third-party packages that integrate with Django need some post-installation
setup to integrate them with your project. Often you will need to add the
package’s Django app to your [`INSTALLED_APPS`](../ref/settings.md#std-setting-INSTALLED_APPS) setting. Some packages
need other changes, like additions to your URLconf (`urls.py`).

Django Debug Toolbar requires several setup steps. Follow them in [its
installation guide](https://django-debug-toolbar.readthedocs.io/en/latest/installation.html).
The steps are not duplicated in this tutorial, because as a third-party
package, it may change separately to Django’s schedule.

Once installed, you should be able to see the DjDT “handle” on the right side
of the browser window when you browse to `http://localhost:8000/admin/`.
Click it to open the debug toolbar and use the tools in each panel. See the
[panels documentation page](https://django-debug-toolbar.readthedocs.io/en/latest/panels.html) for more information on what the panels show.

## Getting help from others

At some point you will run into a problem, for example the
toolbar may not render. When this happens and you’re unable to
resolve the issue yourself, there are options available to you.

1. If the problem is with a specific package, check if there’s a
   troubleshooting of FAQ in the package’s documentation. For example the
   Django Debug Toolbar has a [Tips section](https://django-debug-toolbar.readthedocs.io/en/latest/tips.html) that
   outlines troubleshooting options.
2. Search for similar issues on the package’s issue tracker. Django Debug
   Toolbar’s is [on GitHub](https://github.com/jazzband/django-debug-toolbar/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc).
3. Consult the [Django Forum](https://forum.djangoproject.com/).
4. Join the [Django Discord server](https://discord.gg/xcRH6mN4fa).
5. Join the #Django IRC channel on [Libera.chat](https://libera.chat/).

## Installing other third-party packages

There are many more third-party packages, which you can find using the
fantastic Django resource, [Django Packages](https://djangopackages.org/).

It can be difficult to know what third-party packages you should use. This
depends on your needs and goals. Sometimes it’s fine to use a package that’s
in its alpha state. Other times, you need to know it’s production ready.
[Adam Johnson has a blog post](https://adamj.eu/tech/2021/11/04/the-well-maintained-test/) that outlines
a set of characteristics that qualifies a package as “well maintained”.
Django Packages shows data for some of these characteristics, such as when the
package was last updated.

As Adam points out in his post, when the answer to one of the questions is
“no”, that’s an opportunity to contribute.

## What’s next?

The beginner tutorial ends here. In the meantime, you might want to check out
some pointers on [where to go from here](whatsnext.md).

If you are familiar with Python packaging and interested in learning how to
turn polls into a “reusable app”, check out [Advanced tutorial: How to
write reusable apps](reusable-apps.md).
