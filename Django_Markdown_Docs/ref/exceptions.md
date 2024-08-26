# Django Exceptions

Django raises some of its own exceptions as well as standard Python exceptions.

## Django Core Exceptions

Django core exception classes are defined in `django.core.exceptions`.

### `AppRegistryNotReady`

### *exception* AppRegistryNotReady

This exception is raised when attempting to use models before the [app
loading process](applications.md#app-loading-process), which initializes the ORM, is
complete.

### `ObjectDoesNotExist`

### *exception* ObjectDoesNotExist

The base class for [`Model.DoesNotExist`](models/class.md#django.db.models.Model.DoesNotExist) exceptions. A `try/except` for
`ObjectDoesNotExist` will catch
[`DoesNotExist`](models/class.md#django.db.models.Model.DoesNotExist) exceptions for all models.

See [`get()`](models/querysets.md#django.db.models.query.QuerySet.get).

### `EmptyResultSet`

### *exception* EmptyResultSet

`EmptyResultSet` may be raised during query generation if a query won’t
return any results. Most Django projects won’t encounter this exception,
but it might be useful for implementing custom lookups and expressions.

### `FullResultSet`

### *exception* FullResultSet

`FullResultSet` may be raised during query generation if a query will
match everything. Most Django projects won’t encounter this exception, but
it might be useful for implementing custom lookups and expressions.

### `FieldDoesNotExist`

### *exception* FieldDoesNotExist

The `FieldDoesNotExist` exception is raised by a model’s
`_meta.get_field()` method when the requested field does not exist on the
model or on the model’s parents.

### `MultipleObjectsReturned`

### *exception* MultipleObjectsReturned

The base class for [`Model.MultipleObjectsReturned`](models/class.md#django.db.models.Model.MultipleObjectsReturned) exceptions. A
`try/except` for `MultipleObjectsReturned` will catch
[`MultipleObjectsReturned`](models/class.md#django.db.models.Model.MultipleObjectsReturned) exceptions for all
models.

See [`get()`](models/querysets.md#django.db.models.query.QuerySet.get).

### `SuspiciousOperation`

### *exception* SuspiciousOperation

The [`SuspiciousOperation`](#django.core.exceptions.SuspiciousOperation) exception is raised when a user has
performed an operation that should be considered suspicious from a security
perspective, such as tampering with a session cookie. Subclasses of
`SuspiciousOperation` include:

* `DisallowedHost`
* `DisallowedModelAdminLookup`
* `DisallowedModelAdminToField`
* `DisallowedRedirect`
* `InvalidSessionKey`
* `RequestDataTooBig`
* `SuspiciousFileOperation`
* `SuspiciousMultipartForm`
* `SuspiciousSession`
* `TooManyFieldsSent`
* `TooManyFilesSent`

If a `SuspiciousOperation` exception reaches the ASGI/WSGI handler level
it is logged at the `Error` level and results in
a [`HttpResponseBadRequest`](request-response.md#django.http.HttpResponseBadRequest). See the [logging
documentation](../topics/logging.md) for more information.

### `PermissionDenied`

### *exception* PermissionDenied

The [`PermissionDenied`](#django.core.exceptions.PermissionDenied) exception is raised when a user does not have
permission to perform the action requested.

### `ViewDoesNotExist`

### *exception* ViewDoesNotExist

The [`ViewDoesNotExist`](#django.core.exceptions.ViewDoesNotExist) exception is raised by
[`django.urls`](urlresolvers.md#module-django.urls) when a requested view does not exist.

### `MiddlewareNotUsed`

### *exception* MiddlewareNotUsed

The [`MiddlewareNotUsed`](#django.core.exceptions.MiddlewareNotUsed) exception is raised when a middleware is not
used in the server configuration.

### `ImproperlyConfigured`

### *exception* ImproperlyConfigured

The [`ImproperlyConfigured`](#django.core.exceptions.ImproperlyConfigured) exception is raised when Django is
somehow improperly configured – for example, if a value in `settings.py`
is incorrect or unparseable.

### `FieldError`

### *exception* FieldError

The [`FieldError`](#django.core.exceptions.FieldError) exception is raised when there is a problem with a
model field. This can happen for several reasons:

- A field in a model clashes with a field of the same name from an
  abstract base class
- An infinite loop is caused by ordering
- A keyword cannot be parsed from the filter parameters
- A field cannot be determined from a keyword in the query
  parameters
- A join is not permitted on the specified field
- A field name is invalid
- A query contains invalid order_by arguments

### `ValidationError`

### *exception* ValidationError

The [`ValidationError`](#django.core.exceptions.ValidationError) exception is raised when data fails form or
model field validation. For more information about validation, see
[Form and Field Validation](forms/validation.md),
[Model Field Validation](models/instances.md#validating-objects) and the
[Validator Reference](validators.md).

#### `NON_FIELD_ERRORS`

### NON_FIELD_ERRORS

`ValidationError`s that don’t belong to a particular field in a form
or model are classified as `NON_FIELD_ERRORS`. This constant is used
as a key in dictionaries that otherwise map fields to their respective
list of errors.

### `BadRequest`

### *exception* BadRequest

The [`BadRequest`](#django.core.exceptions.BadRequest) exception is raised when the request cannot be
processed due to a client error. If a `BadRequest` exception reaches the
ASGI/WSGI handler level it results in a
[`HttpResponseBadRequest`](request-response.md#django.http.HttpResponseBadRequest).

### `RequestAborted`

### *exception* RequestAborted

The [`RequestAborted`](#django.core.exceptions.RequestAborted) exception is raised when an HTTP body being read
in by the handler is cut off midstream and the client connection closes,
or when the client does not send data and hits a timeout where the server
closes the connection.

It is internal to the HTTP handler modules and you are unlikely to see
it elsewhere. If you are modifying HTTP handling code, you should raise
this when you encounter an aborted request to make sure the socket is
closed cleanly.

### `SynchronousOnlyOperation`

### *exception* SynchronousOnlyOperation

The [`SynchronousOnlyOperation`](#django.core.exceptions.SynchronousOnlyOperation) exception is raised when code that
is only allowed in synchronous Python code is called from an asynchronous
context (a thread with a running asynchronous event loop). These parts of
Django are generally heavily reliant on thread-safety to function and don’t
work correctly under coroutines sharing the same thread.

If you are trying to call code that is synchronous-only from an
asynchronous thread, then create a synchronous thread and call it in that.
You can accomplish this is with [`asgiref.sync.sync_to_async()`](../topics/async.md#asgiref.sync.sync_to_async).

## URL Resolver exceptions

URL Resolver exceptions are defined in `django.urls`.

### `Resolver404`

### *exception* Resolver404

The [`Resolver404`](#django.urls.Resolver404) exception is raised by
[`resolve()`](urlresolvers.md#django.urls.resolve) if the path passed to `resolve()` doesn’t
map to a view. It’s a subclass of [`django.http.Http404`](../topics/http/views.md#django.http.Http404).

### `NoReverseMatch`

### *exception* NoReverseMatch

The [`NoReverseMatch`](#django.urls.NoReverseMatch) exception is raised by [`django.urls`](urlresolvers.md#module-django.urls) when a
matching URL in your URLconf cannot be identified based on the parameters
supplied.

## Database Exceptions

Database exceptions may be imported from `django.db`.

Django wraps the standard database exceptions so that your Django code has a
guaranteed common implementation of these classes.

### *exception* Error

### *exception* InterfaceError

### *exception* DatabaseError

### *exception* DataError

### *exception* OperationalError

### *exception* IntegrityError

### *exception* InternalError

### *exception* ProgrammingError

### *exception* NotSupportedError

The Django wrappers for database exceptions behave exactly the same as
the underlying database exceptions. See [**PEP 249**](https://peps.python.org/pep-0249/), the Python Database API
Specification v2.0, for further information.

As per [**PEP 3134**](https://peps.python.org/pep-3134/), a `__cause__` attribute is set with the original
(underlying) database exception, allowing access to any additional
information provided.

#### *exception* models.ProtectedError

Raised to prevent deletion of referenced objects when using
[`django.db.models.PROTECT`](models/fields.md#django.db.models.PROTECT). [`models.ProtectedError`](#django.db.models.ProtectedError) is a subclass
of [`IntegrityError`](#django.db.IntegrityError).

#### *exception* models.RestrictedError

Raised to prevent deletion of referenced objects when using
[`django.db.models.RESTRICT`](models/fields.md#django.db.models.RESTRICT). [`models.RestrictedError`](#django.db.models.RestrictedError) is a subclass
of [`IntegrityError`](#django.db.IntegrityError).

## HTTP Exceptions

HTTP exceptions may be imported from `django.http`.

### `UnreadablePostError`

### *exception* UnreadablePostError

[`UnreadablePostError`](#django.http.UnreadablePostError) is raised when a user cancels an upload.

## Sessions Exceptions

Sessions exceptions are defined in `django.contrib.sessions.exceptions`.

### `SessionInterrupted`

### *exception* SessionInterrupted

[`SessionInterrupted`](#django.contrib.sessions.exceptions.SessionInterrupted) is raised when a session is destroyed in a
concurrent request. It’s a subclass of
[`BadRequest`](#django.core.exceptions.BadRequest).

## Transaction Exceptions

Transaction exceptions are defined in `django.db.transaction`.

### `TransactionManagementError`

### *exception* TransactionManagementError

[`TransactionManagementError`](#django.db.transaction.TransactionManagementError) is raised for any and all problems
related to database transactions.

## Testing Framework Exceptions

Exceptions provided by the `django.test` package.

### `RedirectCycleError`

#### *exception* client.RedirectCycleError

[`RedirectCycleError`](#django.test.client.RedirectCycleError) is raised when the test client detects a
loop or an overly long chain of redirects.

## Python Exceptions

Django raises built-in Python exceptions when appropriate as well. See the
Python documentation for further information on the [Built-in Exceptions](https://docs.python.org/3/library/exceptions.html#bltin-exceptions).
