# PostgreSQL specific database constraints

PostgreSQL supports additional data integrity constraints available from the
`django.contrib.postgres.constraints` module. They are added in the model
[`Meta.constraints`](../../models/options.md#django.db.models.Options.constraints) option.

## `ExclusionConstraint`

### *class* ExclusionConstraint(\*, name, expressions, index_type=None, condition=None, deferrable=None, include=None, violation_error_code=None, violation_error_message=None)

Creates an exclusion constraint in the database. Internally, PostgreSQL
implements exclusion constraints using indexes. The default index type is
[GiST](https://www.postgresql.org/docs/current/gist.html). To use them,
you need to activate the [btree_gist extension](https://www.postgresql.org/docs/current/btree-gist.html) on PostgreSQL.
You can install it using the
[`BtreeGistExtension`](operations.md#django.contrib.postgres.operations.BtreeGistExtension) migration
operation.

If you attempt to insert a new row that conflicts with an existing row, an
[`IntegrityError`](../../exceptions.md#django.db.IntegrityError) is raised. Similarly, when update
conflicts with an existing row.

Exclusion constraints are checked during the [model validation](../../models/instances.md#validating-objects).

### `name`

#### ExclusionConstraint.name

See [`BaseConstraint.name`](../../models/constraints.md#django.db.models.BaseConstraint.name).

### `expressions`

#### ExclusionConstraint.expressions

An iterable of 2-tuples. The first element is an expression or string. The
second element is an SQL operator represented as a string. To avoid typos, you
may use [`RangeOperators`](fields.md#django.contrib.postgres.fields.RangeOperators) which maps the
operators with strings. For example:

```default
expressions = [
    ("timespan", RangeOperators.ADJACENT_TO),
    (F("room"), RangeOperators.EQUAL),
]
```

The [`OpClass()`](indexes.md#django.contrib.postgres.indexes.OpClass) expression can
be used to specify a custom [operator class](https://www.postgresql.org/docs/current/indexes-opclass.html) for the constraint expressions.
For example:

```default
expressions = [
    (OpClass("circle", name="circle_ops"), RangeOperators.OVERLAPS),
]
```

creates an exclusion constraint on `circle` using `circle_ops`.

### `index_type`

#### ExclusionConstraint.index_type

The index type of the constraint. Accepted values are `GIST` or `SPGIST`.
Matching is case insensitive. If not provided, the default index type is
`GIST`.

### `condition`

#### ExclusionConstraint.condition

A [`Q`](../../models/querysets.md#django.db.models.Q) object that specifies the condition to restrict
a constraint to a subset of rows. For example,
`condition=Q(cancelled=False)`.

These conditions have the same database restrictions as
[`django.db.models.Index.condition`](../../models/indexes.md#django.db.models.Index.condition).

### `deferrable`

#### ExclusionConstraint.deferrable

Set this parameter to create a deferrable exclusion constraint. Accepted values
are `Deferrable.DEFERRED` or `Deferrable.IMMEDIATE`. For example:

```default
from django.contrib.postgres.constraints import ExclusionConstraint
from django.contrib.postgres.fields import RangeOperators
from django.db.models import Deferrable


ExclusionConstraint(
    name="exclude_overlapping_deferred",
    expressions=[
        ("timespan", RangeOperators.OVERLAPS),
    ],
    deferrable=Deferrable.DEFERRED,
)
```

By default constraints are not deferred. A deferred constraint will not be
enforced until the end of the transaction. An immediate constraint will be
enforced immediately after every command.

#### WARNING
Deferred exclusion constraints may lead to a [performance penalty](https://www.postgresql.org/docs/current/sql-createtable.html#id-1.9.3.85.9.4).

### `include`

#### ExclusionConstraint.include

A list or tuple of the names of the fields to be included in the covering
exclusion constraint as non-key columns. This allows index-only scans to be
used for queries that select only included fields
([`include`](#django.contrib.postgres.constraints.ExclusionConstraint.include)) and filter only by indexed fields
([`expressions`](#django.contrib.postgres.constraints.ExclusionConstraint.expressions)).

`include` is supported for GiST indexes. PostgreSQL 14+ also supports
`include` for SP-GiST indexes.

### `violation_error_code`

#### ExclusionConstraint.violation_error_code

The error code used when `ValidationError` is raised during
[model validation](../../models/instances.md#validating-objects). Defaults to `None`.

### `violation_error_message`

The error message used when `ValidationError` is raised during
[model validation](../../models/instances.md#validating-objects). Defaults to
[`BaseConstraint.violation_error_message`](../../models/constraints.md#django.db.models.BaseConstraint.violation_error_message).

### Examples

The following example restricts overlapping reservations in the same room, not
taking canceled reservations into account:

```default
from django.contrib.postgres.constraints import ExclusionConstraint
from django.contrib.postgres.fields import DateTimeRangeField, RangeOperators
from django.db import models
from django.db.models import Q


class Room(models.Model):
    number = models.IntegerField()


class Reservation(models.Model):
    room = models.ForeignKey("Room", on_delete=models.CASCADE)
    timespan = DateTimeRangeField()
    cancelled = models.BooleanField(default=False)

    class Meta:
        constraints = [
            ExclusionConstraint(
                name="exclude_overlapping_reservations",
                expressions=[
                    ("timespan", RangeOperators.OVERLAPS),
                    ("room", RangeOperators.EQUAL),
                ],
                condition=Q(cancelled=False),
            ),
        ]
```

In case your model defines a range using two fields, instead of the native
PostgreSQL range types, you should write an expression that uses the equivalent
function (e.g. `TsTzRange()`), and use the delimiters for the field. Most
often, the delimiters will be `'[)'`, meaning that the lower bound is
inclusive and the upper bound is exclusive. You may use the
[`RangeBoundary`](fields.md#django.contrib.postgres.fields.RangeBoundary) that provides an
expression mapping for the [range boundaries](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-INCLUSIVITY). For example:

```default
from django.contrib.postgres.constraints import ExclusionConstraint
from django.contrib.postgres.fields import (
    DateTimeRangeField,
    RangeBoundary,
    RangeOperators,
)
from django.db import models
from django.db.models import Func, Q


class TsTzRange(Func):
    function = "TSTZRANGE"
    output_field = DateTimeRangeField()


class Reservation(models.Model):
    room = models.ForeignKey("Room", on_delete=models.CASCADE)
    start = models.DateTimeField()
    end = models.DateTimeField()
    cancelled = models.BooleanField(default=False)

    class Meta:
        constraints = [
            ExclusionConstraint(
                name="exclude_overlapping_reservations",
                expressions=[
                    (
                        TsTzRange("start", "end", RangeBoundary()),
                        RangeOperators.OVERLAPS,
                    ),
                    ("room", RangeOperators.EQUAL),
                ],
                condition=Q(cancelled=False),
            ),
        ]
```
