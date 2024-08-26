# The `File` object

The [`django.core.files`](index.md#module-django.core.files) module and its submodules contain built-in classes
for basic file handling in Django.

## The `File` class

### *class* File(file_object, name=None)

The [`File`](#django.core.files.File) class is a thin wrapper around a Python
[file object](https://docs.python.org/3/glossary.html#term-file-object) with some Django-specific additions.
Internally, Django uses this class when it needs to represent a file.

[`File`](#django.core.files.File) objects have the following attributes and methods:

#### name

The name of the file including the relative path from
[`MEDIA_ROOT`](../settings.md#std-setting-MEDIA_ROOT).

#### size

The size of the file in bytes.

#### file

The underlying [file object](https://docs.python.org/3/glossary.html#term-file-object) that this class wraps.

#### mode

The read/write mode for the file.

#### open(mode=None, \*args, \*\*kwargs)

Open or reopen the file (which also does `File.seek(0)`).
The `mode` argument allows the same values
as Python’s built-in [`open()`](https://docs.python.org/3/library/functions.html#open). `*args` and `**kwargs`
are passed after `mode` to Python’s built-in [`open()`](https://docs.python.org/3/library/functions.html#open).

When reopening a file, `mode` will override whatever mode the file
was originally opened with; `None` means to reopen with the original
mode.

It can be used as a context manager, e.g. `with file.open() as f:`.

#### \_\_iter_\_()

Iterate over the file yielding one line at a time.

#### chunks(chunk_size=None)

Iterate over the file yielding “chunks” of a given size. `chunk_size`
defaults to 64 KB.

This is especially useful with very large files since it allows them to
be streamed off disk and avoids storing the whole file in memory.

#### multiple_chunks(chunk_size=None)

Returns `True` if the file is large enough to require multiple chunks
to access all of its content give some `chunk_size`.

#### close()

Close the file.

In addition to the listed methods, [`File`](#django.core.files.File) exposes
the following attributes and methods of its `file` object:
`encoding`, `fileno`, `flush`, `isatty`, `newlines`, `read`,
`readinto`, `readline`, `readlines`, `seek`, `tell`,
`truncate`, `write`, `writelines`, `readable()`, `writable()`,
and `seekable()`.

## The `ContentFile` class

### *class* ContentFile(content, name=None)

The `ContentFile` class inherits from [`File`](#django.core.files.File),
but unlike [`File`](#django.core.files.File) it operates on string content
(bytes also supported), rather than an actual file. For example:

```default
from django.core.files.base import ContentFile

f1 = ContentFile("esta frase está en español")
f2 = ContentFile(b"these are bytes")
```

## The `ImageFile` class

### *class* ImageFile(file_object, name=None)

Django provides a built-in class specifically for images.
[`django.core.files.images.ImageFile`](#django.core.files.images.ImageFile) inherits all the attributes
and methods of [`File`](#django.core.files.File), and additionally
provides the following:

#### width

Width of the image in pixels.

#### height

Height of the image in pixels.

## Additional methods on files attached to objects

Any [`File`](#django.core.files.File) that is associated with an object (as with `Car.photo`,
below) will also have a couple of extra methods:

#### File.save(name, content, save=True)

Saves a new file with the file name and contents provided. This will not
replace the existing file, but will create a new file and update the object
to point to it. If `save` is `True`, the model’s `save()` method will
be called once the file is saved. That is, these two lines:

```pycon
>>> car.photo.save("myphoto.jpg", content, save=False)
>>> car.save()
```

are equivalent to:

```pycon
>>> car.photo.save("myphoto.jpg", content, save=True)
```

Note that the `content` argument must be an instance of either
[`File`](#django.core.files.File) or of a subclass of [`File`](#django.core.files.File), such as
[`ContentFile`](#django.core.files.base.ContentFile).

#### File.delete(save=True)

Removes the file from the model instance and deletes the underlying file.
If `save` is `True`, the model’s `save()` method will be called once
the file is deleted.
