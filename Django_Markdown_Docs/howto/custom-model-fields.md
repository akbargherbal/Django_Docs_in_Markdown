# How to create custom model fields

## Introduction

The [model reference](../topics/db/models.md) documentation explains how to use
Django’s standard field classes – [`CharField`](../ref/models/fields.md#django.db.models.CharField),
[`DateField`](../ref/models/fields.md#django.db.models.DateField), etc. For many purposes, those classes are
all you’ll need. Sometimes, though, the Django version won’t meet your precise
requirements, or you’ll want to use a field that is entirely different from
those shipped with Django.

Django’s built-in field types don’t cover every possible database column type –
only the common types, such as `VARCHAR` and `INTEGER`. For more obscure
column types, such as geographic polygons or even user-created types such as
[PostgreSQL custom types](https://www.postgresql.org/docs/current/sql-createtype.html), you can define your own Django `Field` subclasses.

Alternatively, you may have a complex Python object that can somehow be
serialized to fit into a standard database column type. This is another case
where a `Field` subclass will help you use your object with your models.

### Our example object

Creating custom fields requires a bit of attention to detail. To make things
easier to follow, we’ll use a consistent example throughout this document:
wrapping a Python object representing the deal of cards in a hand of [Bridge](https://en.wikipedia.org/wiki/Contract_bridge).
Don’t worry, you don’t have to know how to play Bridge to follow this example.
You only need to know that 52 cards are dealt out equally to four players, who
are traditionally called *north*, *east*, *south* and *west*.  Our class looks
something like this:

```default
class Hand:
    """A hand of cards (bridge style)"""

    def __init__(self, north, east, south, west):
        # Input parameters are lists of cards ('Ah', '9s', etc.)
        self.north = north
        self.east = east
        self.south = south
        self.west = west

    # ... (other possibly useful methods omitted) ...
```

This is an ordinary Python class, with nothing Django-specific about it.
We’d like to be able to do things like this in our models (we assume the
`hand` attribute on the model is an instance of `Hand`):

```default
example = MyModel.objects.get(pk=1)
print(example.hand.north)

new_hand = Hand(north, east, south, west)
example.hand = new_hand
example.save()
```

We assign to and retrieve from the `hand` attribute in our model just like
any other Python class. The trick is to tell Django how to handle saving and
loading such an object.

In order to use the `Hand` class in our models, we **do not** have to change
this class at all. This is ideal, because it means you can easily write
model support for existing classes where you cannot change the source code.

#### NOTE
You might only be wanting to take advantage of custom database column
types and deal with the data as standard Python types in your models;
strings, or floats, for example. This case is similar to our `Hand`
example and we’ll note any differences as we go along.

## Background theory

### Database storage

Let’s start with model fields. If you break it down, a model field provides a
way to take a normal Python object – string, boolean, `datetime`, or
something more complex like `Hand` – and convert it to and from a format
that is useful when dealing with the database. (Such a format is also useful
for serialization, but as we’ll see later, that is easier once you have the
database side under control).

Fields in a model must somehow be converted to fit into an existing database
column type. Different databases provide different sets of valid column types,
but the rule is still the same: those are the only types you have to work
with. Anything you want to store in the database must fit into one of
those types.

Normally, you’re either writing a Django field to match a particular database
column type, or you will need a way to convert your data to, say, a string.

For our `Hand` example, we could convert the card data to a string of 104
characters by concatenating all the cards together in a predetermined order –
say, all the *north* cards first, then the *east*, *south* and *west* cards. So
`Hand` objects can be saved to text or character columns in the database.

### What does a field class do?

All of Django’s fields (and when we say *fields* in this document, we always
mean model fields and not [form fields](../ref/forms/fields.md)) are subclasses
of [`django.db.models.Field`](../ref/models/fields.md#django.db.models.Field). Most of the information that Django records
about a field is common to all fields – name, help text, uniqueness and so
forth. Storing all that information is handled by `Field`. We’ll get into the
precise details of what `Field` can do later on; for now, suffice it to say
that everything descends from `Field` and then customizes key pieces of the
class behavior.

It’s important to realize that a Django field class is not what is stored in
your model attributes. The model attributes contain normal Python objects. The
field classes you define in a model are actually stored in the `Meta` class
when the model class is created (the precise details of how this is done are
unimportant here). This is because the field classes aren’t necessary when
you’re just creating and modifying attributes. Instead, they provide the
machinery for converting between the attribute value and what is stored in the
database or sent to the [serializer](../topics/serialization.md).

Keep this in mind when creating your own custom fields. The Django `Field`
subclass you write provides the machinery for converting between your Python
instances and the database/serializer values in various ways (there are
differences between storing a value and using a value for lookups, for
example). If this sounds a bit tricky, don’t worry – it will become clearer in
the examples below. Just remember that you will often end up creating two
classes when you want a custom field:

* The first class is the Python object that your users will manipulate.
  They will assign it to the model attribute, they will read from it for
  displaying purposes, things like that. This is the `Hand` class in our
  example.
* The second class is the `Field` subclass. This is the class that knows
  how to convert your first class back and forth between its permanent
  storage form and the Python form.

## Writing a field subclass

When planning your [`Field`](../ref/models/fields.md#django.db.models.Field) subclass, first give some
thought to which existing [`Field`](../ref/models/fields.md#django.db.models.Field) class your new field
is most similar to. Can you subclass an existing Django field and save yourself
some work? If not, you should subclass the [`Field`](../ref/models/fields.md#django.db.models.Field)
class, from which everything is descended.

Initializing your new field is a matter of separating out any arguments that are
specific to your case from the common arguments and passing the latter to the
`__init__()` method of [`Field`](../ref/models/fields.md#django.db.models.Field) (or your parent
class).

In our example, we’ll call our field `HandField`. (It’s a good idea to call
your [`Field`](../ref/models/fields.md#django.db.models.Field) subclass `<Something>Field`, so it’s
easily identifiable as a [`Field`](../ref/models/fields.md#django.db.models.Field) subclass.) It doesn’t
behave like any existing field, so we’ll subclass directly from
[`Field`](../ref/models/fields.md#django.db.models.Field):

```default
from django.db import models


class HandField(models.Field):
    description = "A hand of cards (bridge style)"

    def __init__(self, *args, **kwargs):
        kwargs["max_length"] = 104
        super().__init__(*args, **kwargs)
```

Our `HandField` accepts most of the standard field options (see the list
below), but we ensure it has a fixed length, since it only needs to hold 52
card values plus their suits; 104 characters in total.

#### NOTE
Many of Django’s model fields accept options that they don’t do anything
with. For example, you can pass both
[`editable`](../ref/models/fields.md#django.db.models.Field.editable) and
[`auto_now`](../ref/models/fields.md#django.db.models.DateField.auto_now) to a
[`django.db.models.DateField`](../ref/models/fields.md#django.db.models.DateField) and it will ignore the
[`editable`](../ref/models/fields.md#django.db.models.Field.editable) parameter
([`auto_now`](../ref/models/fields.md#django.db.models.DateField.auto_now) being set implies
`editable=False`). No error is raised in this case.

This behavior simplifies the field classes, because they don’t need to
check for options that aren’t necessary. They pass all the options to
the parent class and then don’t use them later on. It’s up to you whether
you want your fields to be more strict about the options they select, or to
use the more permissive behavior of the current fields.

The `Field.__init__()` method takes the following parameters:

* [`verbose_name`](../ref/models/fields.md#django.db.models.Field.verbose_name)
* `name`
* [`primary_key`](../ref/models/fields.md#django.db.models.Field.primary_key)
* [`max_length`](../ref/models/fields.md#django.db.models.CharField.max_length)
* [`unique`](../ref/models/fields.md#django.db.models.Field.unique)
* [`blank`](../ref/models/fields.md#django.db.models.Field.blank)
* [`null`](../ref/models/fields.md#django.db.models.Field.null)
* [`db_index`](../ref/models/fields.md#django.db.models.Field.db_index)
* `rel`: Used for related fields (like [`ForeignKey`](../ref/models/fields.md#django.db.models.ForeignKey)). For advanced
  use only.
* [`default`](../ref/models/fields.md#django.db.models.Field.default)
* [`editable`](../ref/models/fields.md#django.db.models.Field.editable)
* `serialize`: If `False`, the field will not be serialized when the model
  is passed to Django’s [serializers](../topics/serialization.md). Defaults to
  `True`.
* [`unique_for_date`](../ref/models/fields.md#django.db.models.Field.unique_for_date)
* [`unique_for_month`](../ref/models/fields.md#django.db.models.Field.unique_for_month)
* [`unique_for_year`](../ref/models/fields.md#django.db.models.Field.unique_for_year)
* [`choices`](../ref/models/fields.md#django.db.models.Field.choices)
* [`help_text`](../ref/models/fields.md#django.db.models.Field.help_text)
* [`db_column`](../ref/models/fields.md#django.db.models.Field.db_column)
* [`db_tablespace`](../ref/models/fields.md#django.db.models.Field.db_tablespace): Only for index creation, if the
  backend supports [tablespaces](../topics/db/tablespaces.md). You can usually
  ignore this option.
* [`auto_created`](../ref/models/fields.md#django.db.models.Field.auto_created): `True` if the field was
  automatically created, as for the [`OneToOneField`](../ref/models/fields.md#django.db.models.OneToOneField)
  used by model inheritance. For advanced use only.

All of the options without an explanation in the above list have the same
meaning they do for normal Django fields. See the [field documentation](../ref/models/fields.md) for examples and details.

<a id="custom-field-deconstruct-method"></a>

### Field deconstruction

The counterpoint to writing your `__init__()` method is writing the
[`deconstruct()`](../ref/models/fields.md#django.db.models.Field.deconstruct) method. It’s used during [model migrations](../topics/migrations.md) to tell Django how to take an instance of your new field
and reduce it to a serialized form - in particular, what arguments to pass to
`__init__()` to recreate it.

If you haven’t added any extra options on top of the field you inherited from,
then there’s no need to write a new `deconstruct()` method. If, however,
you’re changing the arguments passed in `__init__()` (like we are in
`HandField`), you’ll need to supplement the values being passed.

`deconstruct()` returns a tuple of four items: the field’s attribute name,
the full import path of the field class, the positional arguments (as a list),
and the keyword arguments (as a dict). Note this is different from the
`deconstruct()` method [for custom classes](../topics/migrations.md#custom-deconstruct-method)
which returns a tuple of three things.

As a custom field author, you don’t need to care about the first two values;
the base `Field` class has all the code to work out the field’s attribute
name and import path. You do, however, have to care about the positional
and keyword arguments, as these are likely the things you are changing.

For example, in our `HandField` class we’re always forcibly setting
max_length in `__init__()`. The `deconstruct()` method on the base `Field`
class will see this and try to return it in the keyword arguments; thus,
we can drop it from the keyword arguments for readability:

```default
from django.db import models


class HandField(models.Field):
    def __init__(self, *args, **kwargs):
        kwargs["max_length"] = 104
        super().__init__(*args, **kwargs)

    def deconstruct(self):
        name, path, args, kwargs = super().deconstruct()
        del kwargs["max_length"]
        return name, path, args, kwargs
```

If you add a new keyword argument, you need to write code in `deconstruct()`
that puts its value into `kwargs` yourself. You should also omit the value
from `kwargs` when it isn’t necessary to reconstruct the state of the field,
such as when the default value is being used:

```default
from django.db import models


class CommaSepField(models.Field):
    "Implements comma-separated storage of lists"

    def __init__(self, separator=",", *args, **kwargs):
        self.separator = separator
        super().__init__(*args, **kwargs)

    def deconstruct(self):
        name, path, args, kwargs = super().deconstruct()
        # Only include kwarg if it's not the default
        if self.separator != ",":
            kwargs["separator"] = self.separator
        return name, path, args, kwargs
```

More complex examples are beyond the scope of this document, but remember -
for any configuration of your Field instance, `deconstruct()` must return
arguments that you can pass to `__init__` to reconstruct that state.

Pay extra attention if you set new default values for arguments in the
`Field` superclass; you want to make sure they’re always included, rather
than disappearing if they take on the old default value.

In addition, try to avoid returning values as positional arguments; where
possible, return values as keyword arguments for maximum future compatibility.
If you change the names of things more often than their position in the
constructor’s argument list, you might prefer positional, but bear in mind that
people will be reconstructing your field from the serialized version for quite
a while (possibly years), depending how long your migrations live for.

You can see the results of deconstruction by looking in migrations that include
the field, and you can test deconstruction in unit tests by deconstructing and
reconstructing the field:

```default
name, path, args, kwargs = my_field_instance.deconstruct()
new_instance = MyField(*args, **kwargs)
self.assertEqual(my_field_instance.some_attribute, new_instance.some_attribute)
```

<a id="custom-field-non-db-attrs"></a>

### Field attributes not affecting database column definition

You can override `Field.non_db_attrs` to customize attributes of a field that
don’t affect a column definition. It’s used during model migrations to detect
no-op `AlterField` operations.

For example:

```default
class CommaSepField(models.Field):
    @property
    def non_db_attrs(self):
        return super().non_db_attrs + ("separator",)
```

### Changing a custom field’s base class

You can’t change the base class of a custom field because Django won’t detect
the change and make a migration for it. For example, if you start with:

```default
class CustomCharField(models.CharField): ...
```

and then decide that you want to use `TextField` instead, you can’t change
the subclass like this:

```default
class CustomCharField(models.TextField): ...
```

Instead, you must create a new custom field class and update your models to
reference it:

```default
class CustomCharField(models.CharField): ...


class CustomTextField(models.TextField): ...
```

As discussed in [removing fields](../topics/migrations.md#migrations-removing-model-fields), you
must retain the original `CustomCharField` class as long as you have
migrations that reference it.

### Documenting your custom field

As always, you should document your field type, so users will know what it is.
In addition to providing a docstring for it, which is useful for developers,
you can also allow users of the admin app to see a short description of the
field type via the [django.contrib.admindocs](../ref/contrib/admin/admindocs.md) application. To do this provide descriptive
text in a [`description`](../ref/models/fields.md#django.db.models.Field.description) class attribute of your custom field. In
the above example, the description displayed by the `admindocs` application
for a `HandField` will be ‘A hand of cards (bridge style)’.

In the [`django.contrib.admindocs`](../ref/contrib/admin/admindocs.md#module-django.contrib.admindocs) display, the field description is
interpolated with `field.__dict__` which allows the description to
incorporate arguments of the field. For example, the description for
[`CharField`](../ref/models/fields.md#django.db.models.CharField) is:

```default
description = _("String (up to %(max_length)s)")
```

### Useful methods

Once you’ve created your [`Field`](../ref/models/fields.md#django.db.models.Field) subclass, you might
consider overriding a few standard methods, depending on your field’s behavior.
The list of methods below is in approximately decreasing order of importance,
so start from the top.

<a id="custom-database-types"></a>

#### Custom database types

Say you’ve created a PostgreSQL custom type called `mytype`. You can
subclass `Field` and implement the [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type) method, like so:

```default
from django.db import models


class MytypeField(models.Field):
    def db_type(self, connection):
        return "mytype"
```

Once you have `MytypeField`, you can use it in any model, just like any other
`Field` type:

```default
class Person(models.Model):
    name = models.CharField(max_length=80)
    something_else = MytypeField()
```

If you aim to build a database-agnostic application, you should account for
differences in database column types. For example, the date/time column type
in PostgreSQL is called `timestamp`, while the same column in MySQL is called
`datetime`. You can handle this in a [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type) method by
checking the `connection.vendor` attribute. Current built-in vendor names
are: `sqlite`, `postgresql`, `mysql`, and `oracle`.

For example:

```default
class MyDateField(models.Field):
    def db_type(self, connection):
        if connection.vendor == "mysql":
            return "datetime"
        else:
            return "timestamp"
```

The [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type) and [`rel_db_type()`](../ref/models/fields.md#django.db.models.Field.rel_db_type) methods are called by
Django when the framework constructs the `CREATE TABLE` statements for your
application – that is, when you first create your tables. The methods are also
called when constructing a `WHERE` clause that includes the model field –
that is, when you retrieve data using QuerySet methods like `get()`,
`filter()`, and `exclude()` and have the model field as an argument.

Some database column types accept parameters, such as `CHAR(25)`, where the
parameter `25` represents the maximum column length. In cases like these,
it’s more flexible if the parameter is specified in the model rather than being
hard-coded in the `db_type()` method. For example, it wouldn’t make much
sense to have a `CharMaxlength25Field`, shown here:

```default
# This is a silly example of hard-coded parameters.
class CharMaxlength25Field(models.Field):
    def db_type(self, connection):
        return "char(25)"


# In the model:
class MyModel(models.Model):
    # ...
    my_field = CharMaxlength25Field()
```

The better way of doing this would be to make the parameter specifiable at run
time – i.e., when the class is instantiated. To do that, implement
`Field.__init__()`, like so:

```default
# This is a much more flexible example.
class BetterCharField(models.Field):
    def __init__(self, max_length, *args, **kwargs):
        self.max_length = max_length
        super().__init__(*args, **kwargs)

    def db_type(self, connection):
        return "char(%s)" % self.max_length


# In the model:
class MyModel(models.Model):
    # ...
    my_field = BetterCharField(25)
```

Finally, if your column requires truly complex SQL setup, return `None` from
[`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type). This will cause Django’s SQL creation code to skip
over this field. You are then responsible for creating the column in the right
table in some other way, but this gives you a way to tell Django to get out of
the way.

The [`rel_db_type()`](../ref/models/fields.md#django.db.models.Field.rel_db_type) method is called by fields such as `ForeignKey`
and `OneToOneField` that point to another field to determine their database
column data types. For example, if you have an `UnsignedAutoField`, you also
need the foreign keys that point to that field to use the same data type:

```default
# MySQL unsigned integer (range 0 to 4294967295).
class UnsignedAutoField(models.AutoField):
    def db_type(self, connection):
        return "integer UNSIGNED AUTO_INCREMENT"

    def rel_db_type(self, connection):
        return "integer UNSIGNED"
```

<a id="converting-values-to-python-objects"></a>

#### Converting values to Python objects

If your custom [`Field`](../ref/models/fields.md#django.db.models.Field) class deals with data structures that are more
complex than strings, dates, integers, or floats, then you may need to override
[`from_db_value()`](../ref/models/fields.md#django.db.models.Field.from_db_value) and [`to_python()`](../ref/models/fields.md#django.db.models.Field.to_python).

If present for the field subclass, `from_db_value()` will be called in all
circumstances when the data is loaded from the database, including in
aggregates and [`values()`](../ref/models/querysets.md#django.db.models.query.QuerySet.values) calls.

`to_python()` is called by deserialization and during the
[`clean()`](../ref/models/instances.md#django.db.models.Model.clean) method used from forms.

As a general rule, `to_python()` should deal gracefully with any of the
following arguments:

* An instance of the correct type (e.g., `Hand` in our ongoing example).
* A string
* `None` (if the field allows `null=True`)

In our `HandField` class, we’re storing the data as a `VARCHAR` field in
the database, so we need to be able to process strings and `None` in the
`from_db_value()`. In `to_python()`, we need to also handle `Hand`
instances:

```default
import re

from django.core.exceptions import ValidationError
from django.db import models
from django.utils.translation import gettext_lazy as _


def parse_hand(hand_string):
    """Takes a string of cards and splits into a full hand."""
    p1 = re.compile(".{26}")
    p2 = re.compile("..")
    args = [p2.findall(x) for x in p1.findall(hand_string)]
    if len(args) != 4:
        raise ValidationError(_("Invalid input for a Hand instance"))
    return Hand(*args)


class HandField(models.Field):
    # ...

    def from_db_value(self, value, expression, connection):
        if value is None:
            return value
        return parse_hand(value)

    def to_python(self, value):
        if isinstance(value, Hand):
            return value

        if value is None:
            return value

        return parse_hand(value)
```

Notice that we always return a `Hand` instance from these methods. That’s the
Python object type we want to store in the model’s attribute.

For `to_python()`, if anything goes wrong during value conversion, you should
raise a [`ValidationError`](../ref/exceptions.md#django.core.exceptions.ValidationError) exception.

<a id="converting-python-objects-to-query-values"></a>

#### Converting Python objects to query values

Since using a database requires conversion in both ways, if you override
[`from_db_value()`](../ref/models/fields.md#django.db.models.Field.from_db_value) you also have to override
[`get_prep_value()`](../ref/models/fields.md#django.db.models.Field.get_prep_value) to convert Python objects back to query values.

For example:

```default
class HandField(models.Field):
    # ...

    def get_prep_value(self, value):
        return "".join(
            ["".join(l) for l in (value.north, value.east, value.south, value.west)]
        )
```

#### WARNING
If your custom field uses the `CHAR`, `VARCHAR` or `TEXT`
types for MySQL, you must make sure that [`get_prep_value()`](../ref/models/fields.md#django.db.models.Field.get_prep_value)
always returns a string type. MySQL performs flexible and unexpected
matching when a query is performed on these types and the provided
value is an integer, which can cause queries to include unexpected
objects in their results. This problem cannot occur if you always
return a string type from [`get_prep_value()`](../ref/models/fields.md#django.db.models.Field.get_prep_value).

<a id="converting-query-values-to-database-values"></a>

#### Converting query values to database values

Some data types (for example, dates) need to be in a specific format
before they can be used by a database backend.
[`get_db_prep_value()`](../ref/models/fields.md#django.db.models.Field.get_db_prep_value) is the method where those conversions should
be made. The specific connection that will be used for the query is
passed as the `connection` parameter. This allows you to use
backend-specific conversion logic if it is required.

For example, Django uses the following method for its
[`BinaryField`](../ref/models/fields.md#django.db.models.BinaryField):

```default
def get_db_prep_value(self, value, connection, prepared=False):
    value = super().get_db_prep_value(value, connection, prepared)
    if value is not None:
        return connection.Database.Binary(value)
    return value
```

In case your custom field needs a special conversion when being saved that is
not the same as the conversion used for normal query parameters, you can
override [`get_db_prep_save()`](../ref/models/fields.md#django.db.models.Field.get_db_prep_save).

<a id="preprocessing-values-before-saving"></a>

#### Preprocessing values before saving

If you want to preprocess the value just before saving, you can use
[`pre_save()`](../ref/models/fields.md#django.db.models.Field.pre_save). For example, Django’s
[`DateTimeField`](../ref/models/fields.md#django.db.models.DateTimeField) uses this method to set the attribute
correctly in the case of [`auto_now`](../ref/models/fields.md#django.db.models.DateField.auto_now) or
[`auto_now_add`](../ref/models/fields.md#django.db.models.DateField.auto_now_add).

If you do override this method, you must return the value of the attribute at
the end. You should also update the model’s attribute if you make any changes
to the value so that code holding references to the model will always see the
correct value.

<a id="specifying-form-field-for-model-field"></a>

#### Specifying the form field for a model field

To customize the form field used by [`ModelForm`](../topics/forms/modelforms.md#django.forms.ModelForm), you can
override [`formfield()`](../ref/models/fields.md#django.db.models.Field.formfield).

The form field class can be specified via the `form_class` and
`choices_form_class` arguments; the latter is used if the field has choices
specified, the former otherwise. If these arguments are not provided,
[`CharField`](../ref/forms/fields.md#django.forms.CharField) or [`TypedChoiceField`](../ref/forms/fields.md#django.forms.TypedChoiceField)
will be used.

All of the `kwargs` dictionary is passed directly to the form field’s
`__init__()` method. Normally, all you need to do is set up a good default
for the `form_class` (and maybe `choices_form_class`) argument and then
delegate further handling to the parent class. This might require you to write
a custom form field (and even a form widget). See the [forms documentation](../topics/forms/index.md) for information about this.

Continuing our ongoing example, we can write the [`formfield()`](../ref/models/fields.md#django.db.models.Field.formfield) method
as:

```default
class HandField(models.Field):
    # ...

    def formfield(self, **kwargs):
        # This is a fairly standard way to set up some defaults
        # while letting the caller override them.
        defaults = {"form_class": MyFormField}
        defaults.update(kwargs)
        return super().formfield(**defaults)
```

This assumes we’ve imported a `MyFormField` field class (which has its own
default widget). This document doesn’t cover the details of writing custom form
fields.

<a id="emulating-built-in-field-types"></a>

#### Emulating built-in field types

If you have created a [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type) method, you don’t need to worry about
[`get_internal_type()`](../ref/models/fields.md#django.db.models.Field.get_internal_type) – it won’t be used much. Sometimes, though, your
database storage is similar in type to some other field, so you can use that
other field’s logic to create the right column.

For example:

```default
class HandField(models.Field):
    # ...

    def get_internal_type(self):
        return "CharField"
```

No matter which database backend we are using, this will mean that
[`migrate`](../ref/django-admin.md#django-admin-migrate) and other SQL commands create the right column type for
storing a string.

If [`get_internal_type()`](../ref/models/fields.md#django.db.models.Field.get_internal_type) returns a string that is not known to Django for
the database backend you are using – that is, it doesn’t appear in
`django.db.backends.<db_name>.base.DatabaseWrapper.data_types` – the string
will still be used by the serializer, but the default [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type)
method will return `None`. See the documentation of [`db_type()`](../ref/models/fields.md#django.db.models.Field.db_type)
for reasons why this might be useful. Putting a descriptive string in as the
type of the field for the serializer is a useful idea if you’re ever going to
be using the serializer output in some other place, outside of Django.

<a id="converting-model-field-to-serialization"></a>

#### Converting field data for serialization

To customize how the values are serialized by a serializer, you can override
[`value_to_string()`](../ref/models/fields.md#django.db.models.Field.value_to_string). Using [`value_from_object()`](../ref/models/fields.md#django.db.models.Field.value_from_object) is the
best way to get the field’s value prior to serialization. For example, since
`HandField` uses strings for its data storage anyway, we can reuse some
existing conversion code:

```default
class HandField(models.Field):
    # ...

    def value_to_string(self, obj):
        value = self.value_from_object(obj)
        return self.get_prep_value(value)
```

### Some general advice

Writing a custom field can be a tricky process, particularly if you’re doing
complex conversions between your Python types and your database and
serialization formats. Here are a couple of tips to make things go more
smoothly:

1. Look at the existing Django fields (in
   [django/db/models/fields/_\_init_\_.py](https://github.com/django/django/blob/main/django/db/models/fields/__init__.py)) for inspiration. Try to find
   a field that’s similar to what you want and extend it a little bit,
   instead of creating an entirely new field from scratch.
2. Put a `__str__()` method on the class you’re wrapping up as a field. There
   are a lot of places where the default behavior of the field code is to call
   `str()` on the value. (In our examples in this document, `value` would
   be a `Hand` instance, not a `HandField`). So if your `__str__()`
   method automatically converts to the string form of your Python object, you
   can save yourself a lot of work.

## Writing a `FileField` subclass

In addition to the above methods, fields that deal with files have a few other
special requirements which must be taken into account. The majority of the
mechanics provided by `FileField`, such as controlling database storage and
retrieval, can remain unchanged, leaving subclasses to deal with the challenge
of supporting a particular type of file.

Django provides a `File` class, which is used as a proxy to the file’s
contents and operations. This can be subclassed to customize how the file is
accessed, and what methods are available. It lives at
`django.db.models.fields.files`, and its default behavior is explained in the
[file documentation](../ref/files/file.md).

Once a subclass of `File` is created, the new `FileField` subclass must be
told to use it. To do so, assign the new `File` subclass to the special
`attr_class` attribute of the `FileField` subclass.

### A few suggestions

In addition to the above details, there are a few guidelines which can greatly
improve the efficiency and readability of the field’s code.

1. The source for Django’s own `ImageField` (in
   [django/db/models/fields/files.py](https://github.com/django/django/blob/main/django/db/models/fields/files.py)) is a great example of how to
   subclass `FileField` to support a particular type of file, as it
   incorporates all of the techniques described above.
2. Cache file attributes wherever possible. Since files may be stored in
   remote storage systems, retrieving them may cost extra time, or even
   money, that isn’t always necessary. Once a file is retrieved to obtain
   some data about its content, cache as much of that data as possible to
   reduce the number of times the file must be retrieved on subsequent
   calls for that information.
