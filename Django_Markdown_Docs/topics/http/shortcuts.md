# Django shortcut functions

<a id="index-0"></a>

The package `django.shortcuts` collects helper functions and classes that
“span” multiple levels of MVC. In other words, these functions/classes
introduce controlled coupling for convenience’s sake.

## `render()`

### render(request, template_name, context=None, content_type=None, status=None, using=None)

Combines a given template with a given context dictionary and returns an
[`HttpResponse`](../../ref/request-response.md#django.http.HttpResponse) object with that rendered text.

Django does not provide a shortcut function which returns a
[`TemplateResponse`](../../ref/template-response.md#django.template.response.TemplateResponse) because the constructor
of [`TemplateResponse`](../../ref/template-response.md#django.template.response.TemplateResponse) offers the same level
of convenience as [`render()`](#django.shortcuts.render).

### Required arguments

`request`
: The request object used to generate this response.

`template_name`
: The full name of a template to use or sequence of template names. If a
  sequence is given, the first template that exists will be used. See the
  [template loading documentation](../templates.md#template-loading) for more
  information on how templates are found.

### Optional arguments

`context`
: A dictionary of values to add to the template context. By default, this
  is an empty dictionary. If a value in the dictionary is callable, the
  view will call it just before rendering the template.

`content_type`
: The MIME type to use for the resulting document. Defaults to
  `'text/html'`.

`status`
: The status code for the response. Defaults to `200`.

`using`
: The [`NAME`](../../ref/settings.md#std-setting-TEMPLATES-NAME) of a template engine to use for
  loading the template.

### Example

The following example renders the template `myapp/index.html` with the
MIME type *application/xhtml+xml*:

```default
from django.shortcuts import render


def my_view(request):
    # View code here...
    return render(
        request,
        "myapp/index.html",
        {
            "foo": "bar",
        },
        content_type="application/xhtml+xml",
    )
```

This example is equivalent to:

```default
from django.http import HttpResponse
from django.template import loader


def my_view(request):
    # View code here...
    t = loader.get_template("myapp/index.html")
    c = {"foo": "bar"}
    return HttpResponse(t.render(c, request), content_type="application/xhtml+xml")
```

## `redirect()`

### redirect(to, \*args, permanent=False, \*\*kwargs)

Returns an [`HttpResponseRedirect`](../../ref/request-response.md#django.http.HttpResponseRedirect) to the appropriate URL
for the arguments passed.

The arguments could be:

* A model: the model’s [`get_absolute_url()`](../../ref/models/instances.md#django.db.models.Model.get_absolute_url)
  function will be called.
* A view name, possibly with arguments: [`reverse()`](../../ref/urlresolvers.md#django.urls.reverse) will be
  used to reverse-resolve the name.
* An absolute or relative URL, which will be used as-is for the redirect
  location.

By default issues a temporary redirect; pass `permanent=True` to issue a
permanent redirect.

### Examples

You can use the [`redirect()`](#django.shortcuts.redirect) function in a number of ways.

1. By passing some object; that object’s
   [`get_absolute_url()`](../../ref/models/instances.md#django.db.models.Model.get_absolute_url) method will be called
   to figure out the redirect URL:
   ```default
   from django.shortcuts import redirect


   def my_view(request):
       ...
       obj = MyModel.objects.get(...)
       return redirect(obj)
   ```
2. By passing the name of a view and optionally some positional or
   keyword arguments; the URL will be reverse resolved using the
   [`reverse()`](../../ref/urlresolvers.md#django.urls.reverse) method:
   ```default
   def my_view(request):
       ...
       return redirect("some-view-name", foo="bar")
   ```
3. By passing a hardcoded URL to redirect to:
   ```default
   def my_view(request):
       ...
       return redirect("/some/url/")
   ```

   This also works with full URLs:
   ```default
   def my_view(request):
       ...
       return redirect("https://example.com/")
   ```

By default, [`redirect()`](#django.shortcuts.redirect) returns a temporary redirect. All of the above
forms accept a `permanent` argument; if set to `True` a permanent redirect
will be returned:

```default
def my_view(request):
    ...
    obj = MyModel.objects.get(...)
    return redirect(obj, permanent=True)
```

## `get_object_or_404()`

### get_object_or_404(klass, \*args, \*\*kwargs)

### aget_object_or_404(klass, \*args, \*\*kwargs)

*Asynchronous version*: `aget_object_or_404()`

Calls [`get()`](../../ref/models/querysets.md#django.db.models.query.QuerySet.get) on a given model
manager, but it raises [`Http404`](views.md#django.http.Http404) instead of the model’s
[`DoesNotExist`](../../ref/models/class.md#django.db.models.Model.DoesNotExist) exception.

### Arguments

`klass`
: A [`Model`](../../ref/models/instances.md#django.db.models.Model) class,
  a [`Manager`](../db/managers.md#django.db.models.Manager),
  or a [`QuerySet`](../../ref/models/querysets.md#django.db.models.query.QuerySet) instance from which to get
  the object.

`*args`
: [`Q objects`](../../ref/models/querysets.md#django.db.models.Q).

`**kwargs`
: Lookup parameters, which should be in the format accepted by `get()` and
  `filter()`.

### Example

The following example gets the object with the primary key of 1 from
`MyModel`:

```default
from django.shortcuts import get_object_or_404


def my_view(request):
    obj = get_object_or_404(MyModel, pk=1)
```

This example is equivalent to:

```default
from django.http import Http404


def my_view(request):
    try:
        obj = MyModel.objects.get(pk=1)
    except MyModel.DoesNotExist:
        raise Http404("No MyModel matches the given query.")
```

The most common use case is to pass a [`Model`](../../ref/models/instances.md#django.db.models.Model), as
shown above. However, you can also pass a
[`QuerySet`](../../ref/models/querysets.md#django.db.models.query.QuerySet) instance:

```default
queryset = Book.objects.filter(title__startswith="M")
get_object_or_404(queryset, pk=1)
```

The above example is a bit contrived since it’s equivalent to doing:

```default
get_object_or_404(Book, title__startswith="M", pk=1)
```

but it can be useful if you are passed the `queryset` variable from somewhere
else.

Finally, you can also use a [`Manager`](../db/managers.md#django.db.models.Manager). This is useful
for example if you have a
[custom manager](../db/managers.md#custom-managers):

```default
get_object_or_404(Book.dahl_objects, title="Matilda")
```

You can also use
[`related managers`](../../ref/models/relations.md#django.db.models.fields.related.RelatedManager):

```default
author = Author.objects.get(name="Roald Dahl")
get_object_or_404(author.book_set, title="Matilda")
```

Note: As with `get()`, a
[`MultipleObjectsReturned`](../../ref/exceptions.md#django.core.exceptions.MultipleObjectsReturned) exception
will be raised if more than one object is found.

## `get_list_or_404()`

### get_list_or_404(klass, \*args, \*\*kwargs)

### aget_list_or_404(klass, \*args, \*\*kwargs)

*Asynchronous version*: `aget_list_or_404()`

Returns the result of [`filter()`](../../ref/models/querysets.md#django.db.models.query.QuerySet.filter) on
a given model manager cast to a list, raising [`Http404`](views.md#django.http.Http404)
if the resulting list is empty.

### Arguments

`klass`
: A [`Model`](../../ref/models/instances.md#django.db.models.Model), [`Manager`](../db/managers.md#django.db.models.Manager) or
  [`QuerySet`](../../ref/models/querysets.md#django.db.models.query.QuerySet) instance from which to get the
  list.

`*args`
: [`Q objects`](../../ref/models/querysets.md#django.db.models.Q).

`**kwargs`
: Lookup parameters, which should be in the format accepted by `get()` and
  `filter()`.

### Example

The following example gets all published objects from `MyModel`:

```default
from django.shortcuts import get_list_or_404


def my_view(request):
    my_objects = get_list_or_404(MyModel, published=True)
```

This example is equivalent to:

```default
from django.http import Http404


def my_view(request):
    my_objects = list(MyModel.objects.filter(published=True))
    if not my_objects:
        raise Http404("No MyModel matches the given query.")
```
