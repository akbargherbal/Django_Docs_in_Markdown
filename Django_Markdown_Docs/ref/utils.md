# Django Utils

This document covers all stable modules in `django.utils`. Most of the
modules in `django.utils` are designed for internal use and only the
following parts can be considered stable and thus backwards compatible as per
the [internal release deprecation policy](../internals/release-process.md#internal-release-deprecation-policy).

## `django.utils.cache`

This module contains helper functions for controlling HTTP caching. It does so
by managing the `Vary` header of responses. It includes functions to patch
the header of response objects directly and decorators that change functions to
do that header-patching themselves.

For information on the `Vary` header, see [**RFC 9110#section-12.5.5**](https://datatracker.ietf.org/doc/html/rfc9110.html#section-12.5.5).

Essentially, the `Vary` HTTP header defines which headers a cache should take
into account when building its cache key. Requests with the same path but
different header content for headers named in `Vary` need to get different
cache keys to prevent delivery of wrong content.

For example, [internationalization](../topics/i18n/index.md) middleware would
need to distinguish caches by the `Accept-language` header.

### patch_cache_control(response, \*\*kwargs)

This function patches the `Cache-Control` header by adding all keyword
arguments to it. The transformation is as follows:

* All keyword parameter names are turned to lowercase, and underscores
  are converted to hyphens.
* If the value of a parameter is `True` (exactly `True`, not just a
  true value), only the parameter name is added to the header.
* All other parameters are added with their value, after applying
  `str()` to it.

### get_max_age(response)

Returns the max-age from the response Cache-Control header as an integer
(or `None` if it wasn’t found or wasn’t an integer).

### patch_response_headers(response, cache_timeout=None)

Adds some useful headers to the given `HttpResponse` object:

* `Expires`
* `Cache-Control`

Each header is only added if it isn’t already set.

`cache_timeout` is in seconds. The [`CACHE_MIDDLEWARE_SECONDS`](settings.md#std-setting-CACHE_MIDDLEWARE_SECONDS)
setting is used by default.

### add_never_cache_headers(response)

Adds an `Expires` header to the current date/time.

Adds a `Cache-Control: max-age=0, no-cache, no-store, must-revalidate,
private` header to a response to indicate that a page should never be
cached.

Each header is only added if it isn’t already set.

### patch_vary_headers(response, newheaders)

Adds (or updates) the `Vary` header in the given `HttpResponse` object.
`newheaders` is a list of header names that should be in `Vary`. If
headers contains an asterisk, then `Vary` header will consist of a single
asterisk `'*'`, according to [**RFC 9110#section-12.5.5**](https://datatracker.ietf.org/doc/html/rfc9110.html#section-12.5.5). Otherwise,
existing headers in `Vary` aren’t removed.

### get_cache_key(request, key_prefix=None, method='GET', cache=None)

Returns a cache key based on the request path. It can be used in the
request phase because it pulls the list of headers to take into account
from the global path registry and uses those to build a cache key to
check against.

If there is no headerlist stored, the page needs to be rebuilt, so this
function returns `None`.

### learn_cache_key(request, response, cache_timeout=None, key_prefix=None, cache=None)

Learns what headers to take into account for some request path from the
response object. It stores those headers in a global path registry so that
later access to that path will know what headers to take into account
without building the response object itself. The headers are named in
the `Vary` header of the response, but we want to prevent response
generation.

The list of headers to use for cache key generation is stored in the same
cache as the pages themselves. If the cache ages some data out of the
cache, this means that we have to build the response once to get at the
Vary header and so at the list of headers to use for the cache key.

## `django.utils.dateparse`

The functions defined in this module share the following properties:

- They accept strings in ISO 8601 date/time formats (or some close
  alternatives) and return objects from the corresponding classes in Python’s
  [`datetime`](https://docs.python.org/3/library/datetime.html#module-datetime) module.
- They raise [`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError) if their input is well formatted but isn’t a
  valid date or time.
- They return `None` if it isn’t well formatted at all.
- They accept up to picosecond resolution in input, but they truncate it to
  microseconds, since that’s what Python supports.

### parse_date(value)

Parses a string and returns a [`datetime.date`](https://docs.python.org/3/library/datetime.html#datetime.date).

### parse_time(value)

Parses a string and returns a [`datetime.time`](https://docs.python.org/3/library/datetime.html#datetime.time).

UTC offsets aren’t supported; if `value` describes one, the result is
`None`.

### parse_datetime(value)

Parses a string and returns a [`datetime.datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime).

UTC offsets are supported; if `value` describes one, the result’s
`tzinfo` attribute is a [`datetime.timezone`](https://docs.python.org/3/library/datetime.html#datetime.timezone) instance.

### parse_duration(value)

Parses a string and returns a [`datetime.timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta).

Expects data in the format `"DD HH:MM:SS.uuuuuu"`,
`"DD HH:MM:SS,uuuuuu"`,  or as specified by ISO 8601 (e.g.
`P4DT1H15M20S` which is equivalent to `4 1:15:20`) or PostgreSQL’s
day-time interval format (e.g. `3 days 04:05:06`).

## `django.utils.decorators`

### method_decorator(decorator, name='')

Converts a function decorator into a method decorator. It can be used to
decorate methods or classes; in the latter case, `name` is the name
of the method to be decorated and is required.

`decorator` may also be a list or tuple of functions. They are wrapped
in reverse order so that the call order is the order in which the functions
appear in the list/tuple.

See [decorating class based views](../topics/class-based-views/intro.md#id1) for
example usage.

### decorator_from_middleware(middleware_class)

Given a middleware class, returns a view decorator. This lets you use
middleware functionality on a per-view basis. The middleware is created
with no params passed.

It assumes middleware that’s compatible with the old style of Django 1.9
and earlier (having methods like `process_request()`,
`process_exception()`, and `process_response()`).

### decorator_from_middleware_with_args(middleware_class)

Like `decorator_from_middleware`, but returns a function
that accepts the arguments to be passed to the middleware_class.
For example, the [`cache_page()`](../topics/cache.md#django.views.decorators.cache.cache_page)
decorator is created from the `CacheMiddleware` like this:

```default
cache_page = decorator_from_middleware_with_args(CacheMiddleware)


@cache_page(3600)
def my_view(request):
    pass
```

### sync_only_middleware(middleware)

Marks a middleware as [synchronous-only](../topics/http/middleware.md#async-middleware). (The
default in Django, but this allows you to future-proof if the default ever
changes in a future release.)

### async_only_middleware(middleware)

Marks a middleware as [asynchronous-only](../topics/http/middleware.md#async-middleware). Django
will wrap it in an asynchronous event loop when it is called from the WSGI
request path.

### sync_and_async_middleware(middleware)

Marks a middleware as [sync and async compatible](../topics/http/middleware.md#async-middleware),
this allows to avoid converting requests. You must implement detection of
the current request type to use this decorator. See [asynchronous
middleware documentation](../topics/http/middleware.md#async-middleware) for details.

## `django.utils.encoding`

### smart_str(s, encoding='utf-8', strings_only=False, errors='strict')

Returns a `str` object representing arbitrary object `s`. Treats
bytestrings using the `encoding` codec.

If `strings_only` is `True`, don’t convert (some) non-string-like
objects.

### is_protected_type(obj)

Determine if the object instance is of a protected type.

Objects of protected types are preserved as-is when passed to
`force_str(strings_only=True)`.

### force_str(s, encoding='utf-8', strings_only=False, errors='strict')

Similar to `smart_str()`, except that lazy instances are resolved to
strings, rather than kept as lazy objects.

If `strings_only` is `True`, don’t convert (some) non-string-like
objects.

### smart_bytes(s, encoding='utf-8', strings_only=False, errors='strict')

Returns a bytestring version of arbitrary object `s`, encoded as
specified in `encoding`.

If `strings_only` is `True`, don’t convert (some) non-string-like
objects.

### force_bytes(s, encoding='utf-8', strings_only=False, errors='strict')

Similar to `smart_bytes`, except that lazy instances are resolved to
bytestrings, rather than kept as lazy objects.

If `strings_only` is `True`, don’t convert (some) non-string-like
objects.

### iri_to_uri(iri)

Convert an Internationalized Resource Identifier (IRI) portion to a URI
portion that is suitable for inclusion in a URL.

This is the algorithm from section 3.1 of [**RFC 3987#section-3.1**](https://datatracker.ietf.org/doc/html/rfc3987.html#section-3.1), slightly
simplified since the input is assumed to be a string rather than an
arbitrary byte stream.

Takes an IRI (string or UTF-8 bytes) and returns a string containing the
encoded result.

### uri_to_iri(uri)

Converts a Uniform Resource Identifier into an Internationalized Resource
Identifier.

This is an algorithm from section 3.2 of [**RFC 3987#section-3.2**](https://datatracker.ietf.org/doc/html/rfc3987.html#section-3.2).

Takes a URI in ASCII bytes and returns a string containing the encoded
result.

### filepath_to_uri(path)

Convert a file system path to a URI portion that is suitable for inclusion
in a URL. The path is assumed to be either UTF-8 bytes, string, or a
[`Path`](https://docs.python.org/3/library/pathlib.html#pathlib.Path).

This method will encode certain characters that would normally be
recognized as special characters for URIs.  Note that this method does not
encode the ‘ character, as it is a valid character within URIs. See
`encodeURIComponent()` JavaScript function for more details.

Returns an ASCII string containing the encoded result.

### escape_uri_path(path)

Escapes the unsafe characters from the path portion of a Uniform Resource
Identifier (URI).

## `django.utils.feedgenerator`

Sample usage:

```pycon
>>> from django.utils import feedgenerator
>>> feed = feedgenerator.Rss201rev2Feed(
...     title="Poynter E-Media Tidbits",
...     link="https://www.poynter.org/tag/e-media-tidbits/",
...     description="A group blog by the sharpest minds in online media/journalism/publishing.",
...     language="en",
... )
>>> feed.add_item(
...     title="Hello",
...     link="https://www.holovaty.com/test/",
...     description="Testing.",
... )
>>> with open("test.rss", "w") as fp:
...     feed.write(fp, "utf-8")
...
```

For simplifying the selection of a generator use `feedgenerator.DefaultFeed`
which is currently `Rss201rev2Feed`

For definitions of the different versions of RSS, see:
[https://web.archive.org/web/20110718035220/http://diveintomark.org/archives/2004/02/04/incompatible-rss](https://web.archive.org/web/20110718035220/http://diveintomark.org/archives/2004/02/04/incompatible-rss)

### get_tag_uri(url, date)

Creates a TagURI.

See [https://web.archive.org/web/20110514113830/http://diveintomark.org/archives/2004/05/28/howto-atom-id](https://web.archive.org/web/20110514113830/http://diveintomark.org/archives/2004/05/28/howto-atom-id)

### `Stylesheet`

#### Versionadded

### *class* Stylesheet(url, mimetype='', media='screen')

Represents an RSS stylesheet.

#### url

Required argument. The URL where the stylesheet is located.

#### mimetype

An optional string containing the MIME type of the stylesheet. If not
specified, Django will attempt to guess it by using Python’s
[`mimetypes.guess_type()`](https://docs.python.org/3/library/mimetypes.html#mimetypes.guess_type). Use `mimetype=None` if you don’t
want your stylesheet to have a MIME type specified.

#### media

An optional string which will be used as the `media` attribute of
the stylesheet. Defaults to `"screen"`. Use `media=None` if you
don’t want your stylesheet to have a `media` attribute.

### `SyndicationFeed`

### *class* SyndicationFeed

Base class for all syndication feeds. Subclasses should provide
`write()`.

#### \_\_init_\_(title, link, description, language=None, author_email=None, author_name=None, author_link=None, subtitle=None, categories=None, feed_url=None, feed_copyright=None, feed_guid=None, ttl=None, stylesheets=None, \*\*kwargs)

Initialize the feed with the given dictionary of metadata, which applies
to the entire feed.

Any extra keyword arguments you pass to `__init__` will be stored in
`self.feed`.

All parameters should be strings, except for two:

* `categories` should be a sequence of strings.
* `stylesheets` should be a sequence of either strings or
  [`Stylesheet`](#django.utils.feedgenerator.Stylesheet) instances.

#### Versionchanged
The `stylesheets` argument was added.

#### add_item(title, link, description, author_email=None, author_name=None, author_link=None, pubdate=None, comments=None, unique_id=None, categories=(), item_copyright=None, ttl=None, updateddate=None, enclosures=None, \*\*kwargs)

Adds an item to the feed. All args are expected to be strings except
`pubdate` and `updateddate`, which are `datetime.datetime`
objects, and `enclosures`, which is a list of `Enclosure` instances.

#### num_items()

#### root_attributes()

Return extra attributes to place on the root (i.e. feed/channel)
element. Called from `write()`.

#### add_root_elements(handler)

Add elements in the root (i.e. feed/channel) element.
Called from `write()`.

#### add_stylesheets(self, handler)

#### Versionadded

Add stylesheet information to the document.
Called from `write()`.

#### item_attributes(item)

Return extra attributes to place on each item (i.e. item/entry)
element.

#### add_item_elements(handler, item)

Add elements on each item (i.e. item/entry) element.

#### write(outfile, encoding)

Outputs the feed in the given encoding to `outfile`, which is a
file-like object. Subclasses should override this.

#### writeString(encoding)

Returns the feed in the given encoding as a string.

#### latest_post_date()

Returns the latest `pubdate` or `updateddate` for all items in the
feed. If no items have either of these attributes this returns the
current UTC date/time.

### `Enclosure`

### *class* Enclosure

Represents an RSS enclosure

### `RssFeed`

### *class* RssFeed(SyndicationFeed)

### `Rss201rev2Feed`

### *class* Rss201rev2Feed(RssFeed)

Spec: [https://cyber.harvard.edu/rss/rss.html](https://cyber.harvard.edu/rss/rss.html)

### `RssUserland091Feed`

### *class* RssUserland091Feed(RssFeed)

Spec: [http://backend.userland.com/rss091](http://backend.userland.com/rss091)

### `Atom1Feed`

### *class* Atom1Feed(SyndicationFeed)

Spec: [**RFC 4287**](https://datatracker.ietf.org/doc/html/rfc4287.html)

## `django.utils.functional`

### *class* cached_property(func)

The `@cached_property` decorator caches the result of a method with a
single `self` argument as a property. The cached result will persist
as long as the instance does, so if the instance is passed around and the
function subsequently invoked, the cached result will be returned.

Consider a typical case, where a view might need to call a model’s method
to perform some computation, before placing the model instance into the
context, where the template might invoke the method once more:

```default
# the model
class Person(models.Model):
    def friends(self):
        # expensive computation
        ...
        return friends


# in the view:
if person.friends():
    ...
```

And in the template you would have:

```html+django
{% for friend in person.friends %}
```

Here, `friends()` will be called twice. Since the instance `person` in
the view and the template are the same, decorating the `friends()` method
with `@cached_property` can avoid that:

```default
from django.utils.functional import cached_property


class Person(models.Model):
    @cached_property
    def friends(self): ...
```

Note that as the method is now a property, in Python code it will need to
be accessed appropriately:

```default
# in the view:
if person.friends:
    ...
```

The cached value can be treated like an ordinary attribute of the instance:

```default
# clear it, requiring re-computation next time it's called
del person.friends  # or delattr(person, "friends")

# set a value manually, that will persist on the instance until cleared
person.friends = ["Huckleberry Finn", "Tom Sawyer"]
```

Because of the way the [descriptor protocol](https://docs.python.org/3/reference/datamodel.html#descriptor-invocation) works, using `del` (or `delattr`) on a
`cached_property` that hasn’t been accessed raises `AttributeError`.

As well as offering potential performance advantages, `@cached_property`
can ensure that an attribute’s value does not change unexpectedly over the
life of an instance. This could occur with a method whose computation is
based on `datetime.now()`, or if a change were saved to the database by
some other process in the brief interval between subsequent invocations of
a method on the same instance.

You can make cached properties of methods. For example, if you had an
expensive `get_friends()` method and wanted to allow calling it without
retrieving the cached value, you could write:

```default
friends = cached_property(get_friends)
```

While `person.get_friends()` will recompute the friends on each call, the
value of the cached property will persist until you delete it as described
above:

```default
x = person.friends  # calls first time
y = person.get_friends()  # calls again
z = person.friends  # does not call
x is z  # is True
```

### *class* classproperty(method=None)

Similar to [`@classmethod`](https://docs.python.org/3/library/functions.html#classmethod), the `@classproperty`
decorator converts the result of a method with a single `cls` argument
into a property that can be accessed directly from the class.

### keep_lazy(func, \*resultclasses)

Django offers many utility functions (particularly in `django.utils`)
that take a string as their first argument and do something to that string.
These functions are used by template filters as well as directly in other
code.

If you write your own similar functions and deal with translations, you’ll
face the problem of what to do when the first argument is a lazy
translation object. You don’t want to convert it to a string immediately,
because you might be using this function outside of a view (and hence the
current thread’s locale setting will not be correct).

For cases like this, use the `django.utils.functional.keep_lazy()`
decorator. It modifies the function so that *if* it’s called with a lazy
translation as one of its arguments, the function evaluation is delayed
until it needs to be converted to a string.

For example:

```default
from django.utils.functional import keep_lazy, keep_lazy_text


def fancy_utility_function(s, *args, **kwargs):
    # Do some conversion on string 's'
    ...


fancy_utility_function = keep_lazy(str)(fancy_utility_function)


# Or more succinctly:
@keep_lazy(str)
def fancy_utility_function(s, *args, **kwargs): ...
```

The `keep_lazy()` decorator takes a number of extra arguments (`*args`)
specifying the type(s) that the original function can return. A common
use case is to have functions that return text. For these, you can pass the
`str` type to `keep_lazy` (or use the [`keep_lazy_text()`](#django.utils.functional.keep_lazy_text) decorator
described in the next section).

Using this decorator means you can write your function and assume that the
input is a proper string, then add support for lazy translation objects at
the end.

### keep_lazy_text(func)

A shortcut for `keep_lazy(str)(func)`.

If you have a function that returns text and you want to be able to take
lazy arguments while delaying their evaluation, you can use this
decorator:

```default
from django.utils.functional import keep_lazy, keep_lazy_text


# Our previous example was:
@keep_lazy(str)
def fancy_utility_function(s, *args, **kwargs): ...


# Which can be rewritten as:
@keep_lazy_text
def fancy_utility_function(s, *args, **kwargs): ...
```

## `django.utils.html`

Usually you should build up HTML using Django’s templates to make use of its
autoescape mechanism, using the utilities in [`django.utils.safestring`](#module-django.utils.safestring)
where appropriate. This module provides some additional low level utilities for
escaping HTML.

### escape(text)

Returns the given text with ampersands, quotes and angle brackets encoded
for use in HTML. The input is first coerced to a string and the output has
[`mark_safe()`](#django.utils.safestring.mark_safe) applied.

### conditional_escape(text)

Similar to `escape()`, except that it doesn’t operate on preescaped
strings, so it will not double escape.

### format_html(format_string, \*args, \*\*kwargs)

This is similar to [`str.format()`](https://docs.python.org/3/library/stdtypes.html#str.format), except that it is appropriate for
building up HTML fragments. The first argument `format_string` is not
escaped but all other args and kwargs are passed through
[`conditional_escape()`](#django.utils.html.conditional_escape) before being passed to `str.format()`.
Finally, the output has [`mark_safe()`](#django.utils.safestring.mark_safe) applied.

For the case of building up small HTML fragments, this function is to be
preferred over string interpolation using `%` or `str.format()`
directly, because it applies escaping to all arguments - just like the
template system applies escaping by default.

So, instead of writing:

```default
mark_safe(
    "%s <b>%s</b> %s"
    % (
        some_html,
        escape(some_text),
        escape(some_other_text),
    )
)
```

You should instead use:

```default
format_html(
    "{} <b>{}</b> {}",
    mark_safe(some_html),
    some_text,
    some_other_text,
)
```

This has the advantage that you don’t need to apply [`escape()`](#django.utils.html.escape) to each
argument and risk a bug and an XSS vulnerability if you forget one.

Note that although this function uses `str.format()` to do the
interpolation, some of the formatting options provided by `str.format()`
(e.g. number formatting) will not work, since all arguments are passed
through [`conditional_escape()`](#django.utils.html.conditional_escape) which (ultimately) calls
[`force_str()`](#django.utils.encoding.force_str) on the values.

#### Deprecated
Deprecated since version 5.0: Support for calling `format_html()` without passing args or kwargs is
deprecated.

### format_html_join(sep, format_string, args_generator)

A wrapper of [`format_html()`](#django.utils.html.format_html), for the common case of a group of
arguments that need to be formatted using the same format string, and then
joined using `sep`. `sep` is also passed through
[`conditional_escape()`](#django.utils.html.conditional_escape).

`args_generator` should be an iterator that yields arguments to pass to
[`format_html()`](#django.utils.html.format_html), either sequences of positional arguments or mappings of
keyword arguments.

For example, tuples can be used for positional arguments:

```default
format_html_join(
    "\n",
    "<li>{} {}</li>",
    ((u.first_name, u.last_name) for u in users),
)
```

Or dictionaries can be used for keyword arguments:

```default
format_html_join(
    "\n",
    '<li data-id="{id}">{id} {title}</li>',
    ({"id": b.id, "title": b.title} for b in books),
)
```

#### Versionchanged
Support for mappings in `args_generator` was added.

### json_script(value, element_id=None, encoder=None)

Escapes all HTML/XML special characters with their Unicode escapes, so
value is safe for use with JavaScript. Also wraps the escaped JSON in a
`<script>` tag. If the `element_id` parameter is not `None`, the
`<script>` tag is given the passed id. For example:

```pycon
>>> json_script({"hello": "world"}, element_id="hello-data")
'<script id="hello-data" type="application/json">{"hello": "world"}</script>'
```

The `encoder`, which defaults to
[`django.core.serializers.json.DjangoJSONEncoder`](../topics/serialization.md#django.core.serializers.json.DjangoJSONEncoder), will be used to
serialize the data. See [JSON serialization](../topics/serialization.md#serialization-formats-json) for more details about this serializer.

### strip_tags(value)

Tries to remove anything that looks like an HTML tag from the string, that
is anything contained within `<>`.

Absolutely NO guarantee is provided about the resulting string being
HTML safe. So NEVER mark safe the result of a `strip_tag` call without
escaping it first, for example with [`escape()`](#django.utils.html.escape).

For example:

```default
strip_tags(value)
```

If `value` is `"<b>Joel</b> <button>is</button> a <span>slug</span>"`
the return value will be `"Joel is a slug"`.

If you are looking for a more robust solution, consider using a third-party
HTML sanitizing tool.

### html_safe()

The `__html__()` method on a class helps non-Django templates detect
classes whose output doesn’t require HTML escaping.

This decorator defines the `__html__()` method on the decorated class
by wrapping `__str__()` in [`mark_safe()`](#django.utils.safestring.mark_safe).
Ensure the `__str__()` method does indeed return text that doesn’t
require HTML escaping.

## `django.utils.http`

### urlencode(query, doseq=False)

A version of Python’s [`urllib.parse.urlencode()`](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.urlencode) function that can
operate on `MultiValueDict` and non-string values.

### http_date(epoch_seconds=None)

Formats the time to match the [**RFC 1123#section-5.2.14**](https://datatracker.ietf.org/doc/html/rfc1123.html#section-5.2.14) date format as
specified by HTTP [**RFC 9110#section-5.6.7**](https://datatracker.ietf.org/doc/html/rfc9110.html#section-5.6.7).

Accepts a floating point number expressed in seconds since the epoch in
UTC–such as that outputted by `time.time()`. If set to `None`,
defaults to the current time.

Outputs a string in the format `Wdy, DD Mon YYYY HH:MM:SS GMT`.

### content_disposition_header(as_attachment, filename)

Constructs a `Content-Disposition` HTTP header value from the given
`filename` as specified by [**RFC 6266**](https://datatracker.ietf.org/doc/html/rfc6266.html). Returns `None` if
`as_attachment` is `False` and `filename` is `None`, otherwise
returns a string suitable for the `Content-Disposition` HTTP header.

### base36_to_int(s)

Converts a base 36 string to an integer.

### int_to_base36(i)

Converts a positive integer to a base 36 string.

### urlsafe_base64_encode(s)

Encodes a bytestring to a base64 string for use in URLs, stripping any
trailing equal signs.

### urlsafe_base64_decode(s)

Decodes a base64 encoded string, adding back any trailing equal signs that
might have been stripped.

## `django.utils.module_loading`

Functions for working with Python modules.

### import_string(dotted_path)

Imports a dotted module path and returns the attribute/class designated by
the last name in the path. Raises `ImportError` if the import failed. For
example:

```default
from django.utils.module_loading import import_string

ValidationError = import_string("django.core.exceptions.ValidationError")
```

is equivalent to:

```default
from django.core.exceptions import ValidationError
```

## `django.utils.safestring`

Functions and classes for working with “safe strings”: strings that can be
displayed safely without further escaping in HTML. Marking something as a “safe
string” means that the producer of the string has already turned characters
that should not be interpreted by the HTML engine (e.g. ‘<’) into the
appropriate entities.

### *class* SafeString

A `str` subclass that has been specifically marked as “safe” (requires no
further escaping) for HTML output purposes.

### mark_safe(s)

Explicitly mark a string as safe for (HTML) output purposes. The returned
object can be used everywhere a string is appropriate.

Can be called multiple times on a single string.

Can also be used as a decorator.

For building up fragments of HTML, you should normally be using
[`django.utils.html.format_html()`](#django.utils.html.format_html) instead.

String marked safe will become unsafe again if modified. For example:

```pycon
>>> mystr = "<b>Hello World</b>   "
>>> mystr = mark_safe(mystr)
>>> type(mystr)
<class 'django.utils.safestring.SafeString'>

>>> mystr = mystr.strip()  # removing whitespace
>>> type(mystr)
<type 'str'>
```

## `django.utils.text`

### format_lazy(format_string, \*args, \*\*kwargs)

A version of [`str.format()`](https://docs.python.org/3/library/stdtypes.html#str.format) for when `format_string`, `args`,
and/or `kwargs` contain lazy objects. The first argument is the string to
be formatted. For example:

```default
from django.utils.text import format_lazy
from django.utils.translation import pgettext_lazy

urlpatterns = [
    path(
        format_lazy("{person}/<int:pk>/", person=pgettext_lazy("URL", "person")),
        PersonDetailView.as_view(),
    ),
]
```

This example allows translators to translate part of the URL. If “person”
is translated to “persona”, the regular expression will match
`persona/(?P<pk>\d+)/$`, e.g. `persona/5/`.

### slugify(value, allow_unicode=False)

Converts a string to a URL slug by:

1. Converting to ASCII if `allow_unicode` is `False` (the default).
2. Converting to lowercase.
3. Removing characters that aren’t alphanumerics, underscores, hyphens, or
   whitespace.
4. Replacing any whitespace or repeated dashes with single dashes.
5. Removing leading and trailing whitespace, dashes, and underscores.

For example:

```pycon
>>> slugify(" Joel is a slug ")
'joel-is-a-slug'
```

If you want to allow Unicode characters, pass `allow_unicode=True`. For
example:

```pycon
>>> slugify("你好 World", allow_unicode=True)
'你好-world'
```

<a id="time-zone-selection-functions"></a>

## `django.utils.timezone`

### get_fixed_timezone(offset)

Returns a [`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo) instance that represents a time zone
with a fixed offset from UTC.

`offset` is a [`datetime.timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta) or an integer number of
minutes. Use positive values for time zones east of UTC and negative
values for west of UTC.

### get_default_timezone()

Returns a [`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo) instance that represents the
[default time zone](../topics/i18n/timezones.md#default-current-time-zone).

### get_default_timezone_name()

Returns the name of the [default time zone](../topics/i18n/timezones.md#default-current-time-zone).

### get_current_timezone()

Returns a [`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo) instance that represents the
[current time zone](../topics/i18n/timezones.md#default-current-time-zone).

### get_current_timezone_name()

Returns the name of the [current time zone](../topics/i18n/timezones.md#default-current-time-zone).

### activate(timezone)

Sets the [current time zone](../topics/i18n/timezones.md#default-current-time-zone). The
`timezone` argument must be an instance of a [`tzinfo`](https://docs.python.org/3/library/datetime.html#datetime.tzinfo)
subclass or a time zone name.

### deactivate()

Unsets the [current time zone](../topics/i18n/timezones.md#default-current-time-zone).

### override(timezone)

This is a Python context manager that sets the [current time zone](../topics/i18n/timezones.md#default-current-time-zone) on entry with [`activate()`](#django.utils.timezone.activate), and restores
the previously active time zone on exit. If the `timezone` argument is
`None`, the [current time zone](../topics/i18n/timezones.md#default-current-time-zone) is unset
on entry with [`deactivate()`](#django.utils.timezone.deactivate) instead.

`override` is also usable as a function decorator.

### localtime(value=None, timezone=None)

Converts an aware [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) to a different time zone,
by default the [current time zone](../topics/i18n/timezones.md#default-current-time-zone).

When `value` is omitted, it defaults to [`now()`](#django.utils.timezone.now).

This function doesn’t work on naive datetimes; use [`make_aware()`](#django.utils.timezone.make_aware)
instead.

### localdate(value=None, timezone=None)

Uses [`localtime()`](#django.utils.timezone.localtime) to convert an aware [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) to a
[`date()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.date) in a different time zone, by default the
[current time zone](../topics/i18n/timezones.md#default-current-time-zone).

When `value` is omitted, it defaults to [`now()`](#django.utils.timezone.now).

This function doesn’t work on naive datetimes.

### now()

Returns a [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) that represents the
current point in time. Exactly what’s returned depends on the value of
[`USE_TZ`](settings.md#std-setting-USE_TZ):

* If [`USE_TZ`](settings.md#std-setting-USE_TZ) is `False`, this will be a
  [naive](../topics/i18n/timezones.md#naive-vs-aware-datetimes) datetime (i.e. a datetime
  without an associated timezone) that represents the current time
  in the system’s local timezone.
* If [`USE_TZ`](settings.md#std-setting-USE_TZ) is `True`, this will be an
  [aware](../topics/i18n/timezones.md#naive-vs-aware-datetimes) datetime representing the
  current time in UTC. Note that [`now()`](#django.utils.timezone.now) will always return
  times in UTC regardless of the value of [`TIME_ZONE`](settings.md#std-setting-TIME_ZONE);
  you can use [`localtime()`](#django.utils.timezone.localtime) to get the time in the current time zone.

### is_aware(value)

Returns `True` if `value` is aware, `False` if it is naive. This
function assumes that `value` is a [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime).

### is_naive(value)

Returns `True` if `value` is naive, `False` if it is aware. This
function assumes that `value` is a [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime).

### make_aware(value, timezone=None)

Returns an aware [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) that represents the same
point in time as `value` in `timezone`, `value` being a naive
[`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime). If `timezone` is set to `None`, it
defaults to the [current time zone](../topics/i18n/timezones.md#default-current-time-zone).

### make_naive(value, timezone=None)

Returns a naive [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime) that represents in
`timezone`  the same point in time as `value`, `value` being an
aware [`datetime`](https://docs.python.org/3/library/datetime.html#datetime.datetime). If `timezone` is set to `None`, it
defaults to the [current time zone](../topics/i18n/timezones.md#default-current-time-zone).

## `django.utils.translation`

For a complete discussion on the usage of the following see the
[translation documentation](../topics/i18n/translation.md).

### gettext(message)

Translates `message` and returns it as a string.

### pgettext(context, message)

Translates `message` given the `context` and returns it as a string.

For more information, see [Contextual markers](../topics/i18n/translation.md#contextual-markers).

### gettext_lazy(message)

### pgettext_lazy(context, message)

Same as the non-lazy versions above, but using lazy execution.

See [lazy translations documentation](../topics/i18n/translation.md#lazy-translations).

### gettext_noop(message)

Marks strings for translation but doesn’t translate them now. This can be
used to store strings in global variables that should stay in the base
language (because they might be used externally) and will be translated
later.

### ngettext(singular, plural, number)

Translates `singular` and `plural` and returns the appropriate string
based on `number`.

### npgettext(context, singular, plural, number)

Translates `singular` and `plural` and returns the appropriate string
based on `number` and the `context`.

### ngettext_lazy(singular, plural, number)

### npgettext_lazy(context, singular, plural, number)

Same as the non-lazy versions above, but using lazy execution.

See [lazy translations documentation](../topics/i18n/translation.md#lazy-translations).

### activate(language)

Fetches the translation object for a given language and activates it as
the current translation object for the current thread.

### deactivate()

Deactivates the currently active translation object so that further \_ calls
will resolve against the default translation object, again.

### deactivate_all()

Makes the active translation object a `NullTranslations()` instance.
This is useful when we want delayed translations to appear as the original
string for some reason.

### override(language, deactivate=False)

A Python context manager that uses
[`django.utils.translation.activate()`](#django.utils.translation.activate) to fetch the translation object
for a given language, activates it as the translation object for the
current thread and reactivates the previous active language on exit.
Optionally, it can deactivate the temporary translation on exit with
[`django.utils.translation.deactivate()`](#django.utils.translation.deactivate) if the `deactivate` argument
is `True`. If you pass `None` as the language argument, a
`NullTranslations()` instance is activated within the context.

`override` is also usable as a function decorator.

### check_for_language(lang_code)

Checks whether there is a global language file for the given language
code (e.g. ‘fr’, ‘pt_BR’). This is used to decide whether a user-provided
language is available.

### get_language()

Returns the currently selected language code. Returns `None` if
translations are temporarily deactivated (by [`deactivate_all()`](#django.utils.translation.deactivate_all) or
when `None` is passed to [`override()`](#django.utils.translation.override)).

### get_language_bidi()

Returns selected language’s BiDi layout:

* `False` = left-to-right layout
* `True` = right-to-left layout

### get_language_from_request(request, check_path=False)

Analyzes the request to find what language the user wants the system to
show. Only languages listed in settings.LANGUAGES are taken into account.
If the user requests a sublanguage where we have a main language, we send
out the main language.

If `check_path` is `True`, the function first checks the requested URL
for whether its path begins with a language code listed in the
[`LANGUAGES`](settings.md#std-setting-LANGUAGES) setting.

### get_supported_language_variant(lang_code, strict=False)

Returns `lang_code` if it’s in the [`LANGUAGES`](settings.md#std-setting-LANGUAGES) setting, possibly
selecting a more generic variant. For example, `'es'` is returned if
`lang_code` is `'es-ar'` and `'es'` is in [`LANGUAGES`](settings.md#std-setting-LANGUAGES) but
`'es-ar'` isn’t.

`lang_code` has a maximum accepted length of 500 characters. A
[`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError) is raised if `lang_code` exceeds this limit and
`strict` is `True`, or if there is no generic variant and `strict`
is `False`.

If `strict` is `False` (the default), a country-specific variant may
be returned when neither the language code nor its generic variant is found.
For example, if only `'es-co'` is in [`LANGUAGES`](settings.md#std-setting-LANGUAGES), that’s
returned for `lang_code`s like `'es'` and `'es-ar'`. Those matches
aren’t returned if `strict=True`.

Raises [`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError) if nothing is found.

#### Versionchanged
In older versions, `lang_code` values over 500 characters were
processed without raising a [`LookupError`](https://docs.python.org/3/library/exceptions.html#LookupError).

### to_locale(language)

Turns a language name (en-us) into a locale name (en_US).

### templatize(src)

Turns a Django template into something that is understood by `xgettext`.
It does so by translating the Django translation tags into standard
`gettext` function invocations.
