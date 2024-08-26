# Coding style

Please follow these coding standards when writing code for inclusion in Django.

<a id="coding-style-pre-commit"></a>

## Pre-commit checks

[pre-commit](https://pre-commit.com) is a framework for managing pre-commit
hooks. These hooks help to identify simple issues before committing code for
review. By checking for these issues before code review it allows the reviewer
to focus on the change itself, and it can also help to reduce the number of CI
runs.

To use the tool, first install `pre-commit` and then the git hooks:

```console
$ python -m pip install pre-commit
$ pre-commit install
```

On the first commit `pre-commit` will install the hooks, these are
installed in their own environments and will take a short while to
install on the first run. Subsequent checks will be significantly faster.
If an error is found an appropriate error message will be displayed.
If the error was with `black` or `isort` then the tool will go ahead and
fix them for you. Review the changes and re-stage for commit if you are happy
with them.

<a id="coding-style-python"></a>

## Python style

* All files should be formatted using the [black](https://pypi.org/project/black/) auto-formatter. This
  will be run by `pre-commit` if that is configured.
* The project repository includes an `.editorconfig` file. We recommend using
  a text editor with [EditorConfig](https://editorconfig.org/) support to avoid indentation and
  whitespace issues. The Python files use 4 spaces for indentation and the HTML
  files use 2 spaces.
* Unless otherwise specified, follow [**PEP 8**](https://peps.python.org/pep-0008/).

  Use [flake8](https://pypi.org/project/flake8/) to check for problems in this area. Note that our
  `.flake8` file contains some excluded files (deprecated modules we don’t
  care about cleaning up and some third-party code that Django vendors) as well
  as some excluded errors that we don’t consider as gross violations. Remember
  that [**PEP 8**](https://peps.python.org/pep-0008/) is only a guide, so respect the style of the surrounding code
  as a primary goal.

  An exception to [**PEP 8**](https://peps.python.org/pep-0008/) is our rules on line lengths. Don’t limit lines of
  code to 79 characters if it means the code looks significantly uglier or is
  harder to read. We allow up to 88 characters as this is the line length used
  by `black`. This check is included when you run `flake8`. Documentation,
  comments, and docstrings should be wrapped at 79 characters, even though
  [**PEP 8**](https://peps.python.org/pep-0008/) suggests 72.
* String variable interpolation may use
  [%-formatting](https://docs.python.org/3/library/stdtypes.html#old-string-formatting), [f-strings](https://docs.python.org/3/reference/lexical_analysis.html#f-strings), or [`str.format()`](https://docs.python.org/3/library/stdtypes.html#str.format) as appropriate, with the goal of
  maximizing code readability.

  Final judgments of readability are left to the Merger’s discretion. As a
  guide, f-strings should use only plain variable and property access, with
  prior local variable assignment for more complex cases:
  ```default
  # Allowed
  f"hello {user}"
  f"hello {user.name}"
  f"hello {self.user.name}"

  # Disallowed
  f"hello {get_user()}"
  f"you are {user.age * 365.25} days old"

  # Allowed with local variable assignment
  user = get_user()
  f"hello {user}"
  user_days_old = user.age * 365.25
  f"you are {user_days_old} days old"
  ```

  f-strings should not be used for any string that may require translation,
  including error and logging messages. In general `format()` is more
  verbose, so the other formatting methods are preferred.

  Don’t waste time doing unrelated refactoring of existing code to adjust the
  formatting method.
* Avoid use of “we” in comments, e.g. “Loop over” rather than “We loop over”.
* Use underscores, not camelCase, for variable, function and method names
  (i.e. `poll.get_unique_voters()`, not `poll.getUniqueVoters()`).
* Use `InitialCaps` for class names (or for factory functions that
  return classes).
* In docstrings, follow the style of existing docstrings and [**PEP 257**](https://peps.python.org/pep-0257/).
* In tests, use
  [`assertRaisesMessage()`](../../../topics/testing/tools.md#django.test.SimpleTestCase.assertRaisesMessage) and
  [`assertWarnsMessage()`](../../../topics/testing/tools.md#django.test.SimpleTestCase.assertWarnsMessage)
  instead of [`assertRaises()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaises) and
  [`assertWarns()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertWarns) so you can check the
  exception or warning message. Use [`assertRaisesRegex()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertRaisesRegex)
  and [`assertWarnsRegex()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertWarnsRegex) only if you need regular
  expression matching.

  Use [`assertIs(…, True/False)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIs) for testing
  boolean values, rather than [`assertTrue()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertTrue) and
  [`assertFalse()`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertFalse), so you can check the actual boolean
  value, not the truthiness of the expression.
* In test docstrings, state the expected behavior that each test demonstrates.
  Don’t include preambles such as “Tests that” or “Ensures that”.

  Reserve ticket references for obscure issues where the ticket has additional
  details that can’t be easily described in docstrings or comments. Include the
  ticket number at the end of a sentence like this:
  ```default
  def test_foo():
      """
      A test docstring looks like this (#123456).
      """
      ...
  ```

<a id="coding-style-imports"></a>

## Imports

* Use [isort](https://pypi.org/project/isort/) to automate import sorting using the guidelines below.

  Quick start:
  ```console
  $ python -m pip install "isort >= 5.1.0"
  $ isort .
  ```

  This runs `isort` recursively from your current directory, modifying any
  files that don’t conform to the guidelines. If you need to have imports out
  of order (to avoid a circular import, for example) use a comment like this:
  ```default
  import module  # isort:skip
  ```
* Put imports in these groups: future, standard library, third-party libraries,
  other Django components, local Django component, try/excepts. Sort lines in
  each group alphabetically by the full module name. Place all `import module`
  statements before `from module import objects` in each section. Use absolute
  imports for other Django components and relative imports for local components.
* On each line, alphabetize the items with the upper case items grouped before
  the lowercase items.
* Break long lines using parentheses and indent continuation lines by 4 spaces.
  Include a trailing comma after the last import and put the closing
  parenthesis on its own line.

  Use a single blank line between the last import and any module level code,
  and use two blank lines above the first function or class.

  For example (comments are for explanatory purposes only):
  ```python
  # future
  from __future__ import unicode_literals

  # standard library
  import json
  from itertools import chain

  # third-party
  import bcrypt

  # Django
  from django.http import Http404
  from django.http.response import (
      Http404,
      HttpResponse,
      HttpResponseNotAllowed,
      StreamingHttpResponse,
      cookie,
  )

  # local Django
  from .models import LogEntry

  # try/except
  try:
      import yaml
  except ImportError:
      yaml = None

  CONSTANT = "foo"


  class Example: ...
  ```
* Use convenience imports whenever available. For example, do this
  ```default
  from django.views import View
  ```

  instead of:
  ```default
  from django.views.generic.base import View
  ```

## Template style

Follow the below rules in Django template code.

* `{% extends %}` should be the first non-comment line.

  Do this:
  ```html+django
  {% extends "base.html" %}

  {% block content %}
    <h1 class="font-semibold text-xl">
      {{ pages.title }}
    </h1>
  {% endblock content %}
  ```

  Or this:
  ```html+django
  {# This is a comment #}
  {% extends "base.html" %}

  {% block content %}
    <h1 class="font-semibold text-xl">
      {{ pages.title }}
    </h1>
  {% endblock content %}
  ```

  Don’t do this:
  ```html+django
  {% load i18n %}
  {% extends "base.html" %}

  {% block content %}
    <h1 class="font-semibold text-xl">
      {{ pages.title }}
    </h1>
  {% endblock content %}
  ```
* Put exactly one space between `{{`, variable contents, and `}}`.

  Do this:
  ```html+django
  {{ user }}
  ```

  Don’t do this:
  ```html+django
  {{user}}
  ```
* In `{% load ... %}`, list libraries in alphabetical order.

  Do this:
  ```html+django
  {% load i18n l10 tz %}
  ```

  Don’t do this:
  ```html+django
  {% load l10 i18n tz %}
  ```
* Put exactly one space between `{%`, tag contents, and `%}`.

  Do this:
  ```html+django
  {% load humanize %}
  ```

  Don’t do this:
  ```html+django
  {%load humanize%}
  ```
* Put the `{% block %}` tag name in the `{% endblock %}` tag if it is not
  on the same line.

  Do this:
  ```html+django
  {% block header %}

    Code goes here

  {% endblock header %}
  ```

  Don’t do this:
  ```html+django
  {% block header %}

    Code goes here

  {% endblock %}
  ```
* Inside curly braces, separate tokens by single spaces, except for around the
  `.` for attribute access and the `|` for a filter.

  Do this:
  ```html+django
  {% if user.name|lower == "admin" %}
  ```

  Don’t do this:
  ```html+django
  {% if user . name | lower  ==  "admin" %}

  {{ user.name | upper }}
  ```
* Within a template using `{% extends %}`, avoid indenting top-level
  `{% block %}` tags.

  Do this:
  ```html+django
  {% extends "base.html" %}

  {% block content %}
  ```

  Don’t do this:
  ```html+django
  {% extends "base.html" %}

    {% block content %}
    ...
  ```

## View style

* In Django views, the first parameter in a view function should be called
  `request`.

  Do this:
  ```default
  def my_view(request, foo): ...
  ```

  Don’t do this:
  ```default
  def my_view(req, foo): ...
  ```

## Model style

* Field names should be all lowercase, using underscores instead of
  camelCase.

  Do this:
  ```default
  class Person(models.Model):
      first_name = models.CharField(max_length=20)
      last_name = models.CharField(max_length=40)
  ```

  Don’t do this:
  ```default
  class Person(models.Model):
      FirstName = models.CharField(max_length=20)
      Last_Name = models.CharField(max_length=40)
  ```
* The `class Meta` should appear *after* the fields are defined, with
  a single blank line separating the fields and the class definition.

  Do this:
  ```default
  class Person(models.Model):
      first_name = models.CharField(max_length=20)
      last_name = models.CharField(max_length=40)

      class Meta:
          verbose_name_plural = "people"
  ```

  Don’t do this:
  ```default
  class Person(models.Model):
      class Meta:
          verbose_name_plural = "people"

      first_name = models.CharField(max_length=20)
      last_name = models.CharField(max_length=40)
  ```
* The order of model inner classes and standard methods should be as
  follows (noting that these are not all required):
  * All database fields
  * Custom manager attributes
  * `class Meta`
  * `def __str__()`
  * `def save()`
  * `def get_absolute_url()`
  * Any custom methods
* If `choices` is defined for a given model field, define each choice as a
  mapping, with an all-uppercase name as a class attribute on the model.
  Example:
  ```default
  class MyModel(models.Model):
      DIRECTION_UP = "U"
      DIRECTION_DOWN = "D"
      DIRECTION_CHOICES = {
          DIRECTION_UP: "Up",
          DIRECTION_DOWN: "Down",
      }
  ```

  Alternatively, consider using [Enumeration types](../../../ref/models/fields.md#field-choices-enum-types):
  ```default
  class MyModel(models.Model):
      class Direction(models.TextChoices):
          UP = "U", "Up"
          DOWN = "D", "Down"
  ```

## Use of `django.conf.settings`

Modules should not in general use settings stored in `django.conf.settings`
at the top level (i.e. evaluated when the module is imported). The explanation
for this is as follows:

Manual configuration of settings (i.e. not relying on the
[`DJANGO_SETTINGS_MODULE`](../../../topics/settings.md#envvar-DJANGO_SETTINGS_MODULE) environment variable) is allowed and possible
as follows:

```default
from django.conf import settings

settings.configure({}, SOME_SETTING="foo")
```

However, if any setting is accessed before the `settings.configure` line,
this will not work. (Internally, `settings` is a `LazyObject` which
configures itself automatically when the settings are accessed if it has not
already been configured).

So, if there is a module containing some code as follows:

```default
from django.conf import settings
from django.urls import get_callable

default_foo_view = get_callable(settings.FOO_VIEW)
```

…then importing this module will cause the settings object to be configured.
That means that the ability for third parties to import the module at the top
level is incompatible with the ability to configure the settings object
manually, or makes it very difficult in some circumstances.

Instead of the above code, a level of laziness or indirection must be used,
such as `django.utils.functional.LazyObject`,
`django.utils.functional.lazy()` or `lambda`.

## Miscellaneous

* Mark all strings for internationalization; see the [i18n
  documentation](../../../topics/i18n/index.md) for details.
* Remove `import` statements that are no longer used when you change code.
  [flake8](https://pypi.org/project/flake8/) will identify these imports for you. If an unused import needs
  to remain for backwards-compatibility, mark the end of with `# NOQA` to
  silence the flake8 warning.
* Systematically remove all trailing whitespaces from your code as those
  add unnecessary bytes, add visual clutter to the patches and can also
  occasionally cause unnecessary merge conflicts. Some IDE’s can be
  configured to automatically remove them and most VCS tools can be set to
  highlight them in diff outputs.
* Please don’t put your name in the code you contribute. Our policy is to
  keep contributors’ names in the `AUTHORS` file distributed with Django
  – not scattered throughout the codebase itself. Feel free to include a
  change to the `AUTHORS` file in your patch if you make more than a
  single trivial change.

## JavaScript style

For details about the JavaScript code style used by Django, see
[JavaScript code](javascript.md).
