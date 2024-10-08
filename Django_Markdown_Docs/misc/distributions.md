# Third-party distributions of Django

Many third-party distributors are now providing versions of Django integrated
with their package-management systems. These can make installation and upgrading
much easier for users of Django since the integration includes the ability to
automatically install dependencies (like database adapters) that Django
requires.

Typically, these packages are based on the latest stable release of Django, so
if you want to use the development version of Django you’ll need to follow the
instructions for [installing the development version](../topics/install.md#installing-development-version) from our Git repository.

If you’re using Linux or a Unix installation, such as OpenSolaris,
check with your distributor to see if they already package Django. If
you’re using a Linux distro and don’t know how to find out if a package
is available, then now is a good time to learn.  The Django Wiki contains
a list of [Third Party Distributions](https://code.djangoproject.com/wiki/Distributions) to help you out.

## For distributors

If you’d like to package Django for distribution, we’d be happy to help out!
Please join the [django-developers](../internals/mailing-lists.md#django-developers-mailing-list) mailing list and introduce yourself.

We also encourage all distributors to subscribe to the [django-announce](../internals/mailing-lists.md#django-announce-mailing-list) mailing
list, which is a (very) low-traffic list for announcing new releases of Django
and important bugfixes.
