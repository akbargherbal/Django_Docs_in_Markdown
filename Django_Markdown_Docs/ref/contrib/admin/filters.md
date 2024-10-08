<a id="modeladmin-list-filters"></a>

# `ModelAdmin` List Filters

`ModelAdmin` classes can define list filters that appear in the right sidebar
of the change list page of the admin, as illustrated in the following
screenshot:

![image](ref/contrib/admin/_images/list_filter.png)

To activate per-field filtering, set [`ModelAdmin.list_filter`](index.md#django.contrib.admin.ModelAdmin.list_filter) to a list
or tuple of elements, where each element is one of the following types:

- A field name.
- A subclass of `django.contrib.admin.SimpleListFilter`.
- A 2-tuple containing a field name and a subclass of
  `django.contrib.admin.FieldListFilter`.

See the examples below for discussion of each of these options for defining
`list_filter`.

## Using a field name

The simplest option is to specify the required field names from your model.

Each specified field should be either a `BooleanField`, `CharField`,
`DateField`, `DateTimeField`, `IntegerField`, `ForeignKey` or
`ManyToManyField`, for example:

```default
class PersonAdmin(admin.ModelAdmin):
    list_filter = ["is_staff", "company"]
```

Field names in `list_filter` can also span relations
using the `__` lookup, for example:

```default
class PersonAdmin(admin.UserAdmin):
    list_filter = ["company__name"]
```

## Using a `SimpleListFilter`

For custom filtering, you can define your own list filter by subclassing
`django.contrib.admin.SimpleListFilter`. You need to provide the `title`
and `parameter_name` attributes, and override the `lookups` and
`queryset` methods, e.g.:

```default
from datetime import date

from django.contrib import admin
from django.utils.translation import gettext_lazy as _


class DecadeBornListFilter(admin.SimpleListFilter):
    # Human-readable title which will be displayed in the
    # right admin sidebar just above the filter options.
    title = _("decade born")

    # Parameter for the filter that will be used in the URL query.
    parameter_name = "decade"

    def lookups(self, request, model_admin):
        """
        Returns a list of tuples. The first element in each
        tuple is the coded value for the option that will
        appear in the URL query. The second element is the
        human-readable name for the option that will appear
        in the right sidebar.
        """
        return [
            ("80s", _("in the eighties")),
            ("90s", _("in the nineties")),
        ]

    def queryset(self, request, queryset):
        """
        Returns the filtered queryset based on the value
        provided in the query string and retrievable via
        `self.value()`.
        """
        # Compare the requested value (either '80s' or '90s')
        # to decide how to filter the queryset.
        if self.value() == "80s":
            return queryset.filter(
                birthday__gte=date(1980, 1, 1),
                birthday__lte=date(1989, 12, 31),
            )
        if self.value() == "90s":
            return queryset.filter(
                birthday__gte=date(1990, 1, 1),
                birthday__lte=date(1999, 12, 31),
            )


class PersonAdmin(admin.ModelAdmin):
    list_filter = [DecadeBornListFilter]
```

#### NOTE
As a convenience, the `HttpRequest` object is passed to the `lookups`
and `queryset` methods, for example:

```default
class AuthDecadeBornListFilter(DecadeBornListFilter):
    def lookups(self, request, model_admin):
        if request.user.is_superuser:
            return super().lookups(request, model_admin)

    def queryset(self, request, queryset):
        if request.user.is_superuser:
            return super().queryset(request, queryset)
```

Also as a convenience, the `ModelAdmin` object is passed to the
`lookups` method, for example if you want to base the lookups on the
available data:

```default
class AdvancedDecadeBornListFilter(DecadeBornListFilter):
    def lookups(self, request, model_admin):
        """
        Only show the lookups if there actually is
        anyone born in the corresponding decades.
        """
        qs = model_admin.get_queryset(request)
        if qs.filter(
            birthday__gte=date(1980, 1, 1),
            birthday__lte=date(1989, 12, 31),
        ).exists():
            yield ("80s", _("in the eighties"))
        if qs.filter(
            birthday__gte=date(1990, 1, 1),
            birthday__lte=date(1999, 12, 31),
        ).exists():
            yield ("90s", _("in the nineties"))
```

## Using a field name and an explicit `FieldListFilter`

Finally, if you wish to specify an explicit filter type to use with a field you
may provide a `list_filter` item as a 2-tuple, where the first element is a
field name and the second element is a class inheriting from
`django.contrib.admin.FieldListFilter`, for example:

```default
class PersonAdmin(admin.ModelAdmin):
    list_filter = [
        ("is_staff", admin.BooleanFieldListFilter),
    ]
```

Here the `is_staff` field will use the `BooleanFieldListFilter`. Specifying
only the field name, fields will automatically use the appropriate filter for
most cases, but this format allows you to control the filter used.

The following examples show available filter classes that you need to opt-in
to use.

You can limit the choices of a related model to the objects involved in
that relation using `RelatedOnlyFieldListFilter`:

```default
class BookAdmin(admin.ModelAdmin):
    list_filter = [
        ("author", admin.RelatedOnlyFieldListFilter),
    ]
```

Assuming `author` is a `ForeignKey` to a `User` model, this will
limit the `list_filter` choices to the users who have written a book,
instead of listing all users.

You can filter empty values using `EmptyFieldListFilter`, which can
filter on both empty strings and nulls, depending on what the field
allows to store:

```default
class BookAdmin(admin.ModelAdmin):
    list_filter = [
        ("title", admin.EmptyFieldListFilter),
    ]
```

By defining a filter using the `__in` lookup, it is possible to filter for
any of a group of values. You need to override the `expected_parameters`
method, and the specify the `lookup_kwargs` attribute with the appropriate
field name. By default, multiple values in the query string will be separated
with commas, but this can be customized via the `list_separator` attribute.
The following example shows such a filter using the vertical-pipe character as
the separator:

```default
class FilterWithCustomSeparator(admin.FieldListFilter):
    # custom list separator that should be used to separate values.
    list_separator = "|"

    def __init__(self, field, request, params, model, model_admin, field_path):
        self.lookup_kwarg = "%s__in" % field_path
        super().__init__(field, request, params, model, model_admin, field_path)

    def expected_parameters(self):
        return [self.lookup_kwarg]
```

#### NOTE
The [`GenericForeignKey`](../contenttypes.md#django.contrib.contenttypes.fields.GenericForeignKey) field is
not supported.

List filters typically appear only if the filter has more than one choice. A
filter’s `has_output()` method controls whether or not it appears.

It is possible to specify a custom template for rendering a list filter:

```default
class FilterWithCustomTemplate(admin.SimpleListFilter):
    template = "custom_template.html"
```

See the default template provided by Django (`admin/filter.html`) for a
concrete example.

<a id="facet-filters"></a>

## Facets

By default, counts for each filter, known as facets, can be shown by toggling
on via the admin UI. These counts will update according to the currently
applied filters. See [`ModelAdmin.show_facets`](index.md#django.contrib.admin.ModelAdmin.show_facets) for more details.
