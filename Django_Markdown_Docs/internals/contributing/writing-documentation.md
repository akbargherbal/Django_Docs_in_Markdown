# Writing documentation

We place high importance on the consistency and readability of documentation.
After all, Django was created in a journalism environment! So we treat our
documentation like we treat our code: we aim to improve it as often as
possible.

Documentation changes generally come in two forms:

* General improvements: typo corrections, error fixes and better
  explanations through clearer writing and more examples.
* New features: documentation of features that have been added to the
  framework since the last release.

This section explains how writers can craft their documentation changes
in the most useful and least error-prone ways.

## The Django documentation process

Though Django’s documentation is intended to be read as HTML at
[https://docs.djangoproject.com/](https://docs.djangoproject.com/), we edit it as a collection of plain text files
written in the reStructuredText markup language for maximum flexibility.

We work from the development version of the repository because it has the
latest-and-greatest documentation, just as it has the latest-and-greatest code.

We also backport documentation fixes and improvements, at the discretion of the
merger, to the last release branch. This is because it’s advantageous to
have the docs for the last release be up-to-date and correct (see
[Differences between versions](../../intro/whatsnext.md#differences-between-doc-versions)).

Django’s documentation uses the [Sphinx](https://www.sphinx-doc.org/) documentation system, which in turn
is based on [docutils](https://docutils.sourceforge.io/). The basic idea is that lightly-formatted plain-text
documentation is transformed into HTML, PDF, and any other output format.

Sphinx includes a `sphinx-build` command for turning reStructuredText into
other formats, e.g., HTML and PDF. This command is configurable, but the Django
documentation includes a `Makefile` that provides a shorter `make html`
command.

## How the documentation is organized

The documentation is organized into several categories:

* [Tutorials](../../intro/index.md) take the reader by the hand through a series
  of steps to create something.

  The important thing in a tutorial is to help the reader achieve something
  useful, preferably as early as possible, in order to give them confidence.

  Explain the nature of the problem we’re solving, so that the reader
  understands what we’re trying to achieve. Don’t feel that you need to begin
  with explanations of how things work - what matters is what the reader does,
  not what you explain. It can be helpful to refer back to what you’ve done and
  explain afterward.
* [Topic guides](../../topics/index.md) aim to explain a concept or subject at a
  fairly high level.

  Link to reference material rather than repeat it. Use examples and don’t be
  reluctant to explain things that seem very basic to you - it might be the
  explanation someone else needs.

  Providing background context helps a newcomer connect the topic to things
  that they already know.
* [Reference guides](../../ref/index.md) contain technical references for APIs.
  They describe the functioning of Django’s internal machinery and instruct in
  its use.

  Keep reference material tightly focused on the subject. Assume that the
  reader already understands the basic concepts involved but needs to know or
  be reminded of how Django does it.

  Reference guides aren’t the place for general explanation. If you find
  yourself explaining basic concepts, you may want to move that material to a
  topic guide.
* [How-to guides](../../howto/index.md) are recipes that take the reader through
  steps in key subjects.

  What matters most in a how-to guide is what a user wants to achieve.
  A how-to should always be result-oriented rather than focused on internal
  details of how Django implements whatever is being discussed.

  These guides are more advanced than tutorials and assume some knowledge about
  how Django works. Assume that the reader has followed the tutorials and don’t
  hesitate to refer the reader back to the appropriate tutorial rather than
  repeat the same material.

## How to start contributing documentation

### Clone the Django repository to your local machine

If you’d like to start contributing to our docs, get the development version of
Django from the source code repository (see
[Installing the development version](../../topics/install.md#installing-development-version)):

```console
$ git clone https://github.com/django/django.git
```

If you’re planning to submit these changes, you might find it useful to make a
fork of the Django repository and clone this fork instead.

### Set up a virtual environment and install dependencies

Create and activate a virtual environment, then install the dependencies:

```shell
$ python -m venv .venv
$ source .venv/bin/activate
$ python -m pip install -r docs/requirements.txt
```

### Build the documentation locally

We can build HTML output from the `docs` directory:

```console
$ cd docs
$ make html
```

Your locally-built documentation will be accessible at
`_build/html/index.html` and it can be viewed in any web browser, though it
will be themed differently than the documentation at
[docs.djangoproject.com](https://docs.djangoproject.com/). This is OK! If
your changes look good on your local machine, they’ll look good on the website.

### Making edits to the documentation

The source files are `.txt` files located in the `docs/` directory.

These files are written in the reStructuredText markup language. To learn the
markup, see the [reStructuredText reference](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html#rst-index).

To edit this page, for example, we would edit the file
[docs/internals/contributing/writing-documentation.txt](https://github.com/django/django/blob/main/docs/internals/contributing/writing-documentation.txt) and rebuild the
HTML with `make html`.

<a id="documentation-spelling-check"></a>

### Spelling check

Before you commit your docs, it’s a good idea to run the spelling checker.
You’ll need to install [sphinxcontrib-spelling](https://pypi.org/project/sphinxcontrib-spelling/) first. Then from the
`docs` directory, run `make spelling`. Wrong words (if any) along with the
file and line number where they occur will be saved to
`_build/spelling/output.txt`.

If you encounter false-positives (error output that actually is correct), do
one of the following:

* Surround inline code or brand/technology names with double grave accents
  (\`\`).
* Find synonyms that the spell checker recognizes.
* If, and only if, you are sure the word you are using is correct - add it
  to `docs/spelling_wordlist` (please keep the list in alphabetical order).

<a id="documentation-link-check"></a>

### Link check

Links in documentation can become broken or changed such that they are no
longer the canonical link. Sphinx provides a builder that can check whether the
links in the documentation are working. From the `docs` directory, run `make
linkcheck`. Output is printed to the terminal, but can also be found in
`_build/linkcheck/output.txt` and `_build/linkcheck/output.json`.

Entries that have a status of “working” are fine, those that are “unchecked” or
“ignored” have been skipped because they either cannot be checked or have
matched ignore rules in the configuration.

Entries that have a status of “broken” need to be fixed. Those that have a
status of “redirected” may need to be updated to point to the canonical
location, e.g. the scheme has changed `http://` → `https://`. In certain
cases, we do not want to update a “redirected” link, e.g. a rewrite to always
point to the latest or stable version of the documentation, e.g. `/en/stable/` →
`/en/3.2/`.

## Writing style

When using pronouns in reference to a hypothetical person, such as “a user with
a session cookie”, gender-neutral pronouns (they/their/them) should be used.
Instead of:

* he or she… use they.
* him or her… use them.
* his or her… use their.
* his or hers… use theirs.
* himself or herself… use themselves.

Try to avoid using words that minimize the difficulty involved in a task or
operation, such as “easily”, “simply”, “just”, “merely”, “straightforward”, and
so on. People’s experience may not match your expectations, and they may become
frustrated when they do not find a step as “straightforward” or “simple” as it
is implied to be.

## Commonly used terms

Here are some style guidelines on commonly used terms throughout the
documentation:

* **Django** – when referring to the framework, capitalize Django. It is
  lowercase only in Python code and in the djangoproject.com logo.
* **email** – no hyphen.
* **HTTP** – the expected pronunciation is “Aitch Tee Tee Pee” and therefore
  should be preceded by “an” and not “a”.
* **MySQL**, **PostgreSQL**, **SQLite**
* **SQL** – when referring to SQL, the expected pronunciation should be
  “Ess Queue Ell” and not “sequel”. Thus in a phrase like “Returns an
  SQL expression”, “SQL” should be preceded by “an” and not “a”.
* **Python** – when referring to the language, capitalize Python.
* **realize**, **customize**, **initialize**, etc. – use the American
  “ize” suffix, not “ise.”
* **subclass** – it’s a single word without a hyphen, both as a verb
  (“subclass that model”) and as a noun (“create a subclass”).
* **the web**, **web framework** – it’s not capitalized.
* **website** – use one word, without capitalization.

## Django-specific terminology

* **model** – it’s not capitalized.
* **template** – it’s not capitalized.
* **URLconf** – use three capitalized letters, with no space before
  “conf.”
* **view** – it’s not capitalized.

## Guidelines for reStructuredText files

These guidelines regulate the format of our reST (reStructuredText)
documentation:

* In section titles, capitalize only initial words and proper nouns.
* Wrap the documentation at 80 characters wide, unless a code example
  is significantly less readable when split over two lines, or for another
  good reason.
* The main thing to keep in mind as you write and edit docs is that the
  more semantic markup you can add the better. So:
  ```rst
  Add ``django.contrib.auth`` to your ``INSTALLED_APPS``...
  ```

  Isn’t nearly as helpful as:
  ```rst
  Add :mod:`django.contrib.auth` to your :setting:`INSTALLED_APPS`...
  ```

  This is because Sphinx will generate proper links for the latter, which
  greatly helps readers.

  You can prefix the target with a `~` (that’s a tilde) to get only the
  “last bit” of that path. So `:mod:`~django.contrib.auth`` will
  display a link with the title “auth”.
* All Python code blocks should be formatted using the [blacken-docs](https://pypi.org/project/blacken-docs/)
  auto-formatter. This will be run by `pre-commit` if that is configured.
* Use [`intersphinx`](https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html#module-sphinx.ext.intersphinx) to reference Python’s and Sphinx’
  documentation.
* Add `.. code-block:: <lang>` to literal blocks so that they get
  highlighted. Prefer relying on automatic highlighting using `::`
  (two colons). This has the benefit that if the code contains some invalid
  syntax, it won’t be highlighted. Adding `.. code-block:: python`, for
  example, will force highlighting despite invalid syntax.
* To improve readability, use `.. admonition:: Descriptive title` rather than
  `.. note::`. Use these boxes sparingly.
* Use these heading styles:
  ```rst
  ===
  One
  ===

  Two
  ===

  Three
  -----

  Four
  ~~~~

  Five
  ^^^^
  ```
* Use [`:rfc:`](https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html#role-rfc) to reference RFC and try to link to the relevant
  section if possible. For example, use `:rfc:`2324#section-2.3.2`` or
  `:rfc:`Custom link text <2324#section-2.3.2>``.
* Use [`:pep:`](https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html#role-pep) to reference a Python Enhancement Proposal (PEP)
  and try to link to the relevant section if possible. For example, use
  `:pep:`20#easter-egg`` or `:pep:`Easter Egg <20#easter-egg>``.
* Use [`:mimetype:`](https://www.sphinx-doc.org/en/master/usage/restructuredtext/roles.html#role-mimetype) to refer to a MIME Type unless the value
  is quoted for a code example.
* Use [`:envvar:`](https://www.sphinx-doc.org/en/master/usage/referencing.html#role-envvar) to refer to an environment variable. You may
  also need to define a reference to the documentation for that environment
  variable using [`.. envvar::`](https://www.sphinx-doc.org/en/master/usage/domains/standard.html#directive-envvar).

## Django-specific markup

Besides [Sphinx’s built-in markup](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html#rst-index), Django’s docs
define some extra description units:

* Settings:
  ```rst
  .. setting:: INSTALLED_APPS
  ```

  To link to a setting, use `:setting:`INSTALLED_APPS``.
* Template tags:
  ```rst
  .. templatetag:: regroup
  ```

  To link, use `:ttag:`regroup``.
* Template filters:
  ```rst
  .. templatefilter:: linebreaksbr
  ```

  To link, use `:tfilter:`linebreaksbr``.
* Field lookups (i.e. `Foo.objects.filter(bar__exact=whatever)`):
  ```rst
  .. fieldlookup:: exact
  ```

  To link, use `:lookup:`exact``.
* `django-admin` commands:
  ```rst
  .. django-admin:: migrate
  ```

  To link, use `:djadmin:`migrate``.
* `django-admin` command-line options:
  ```rst
  .. django-admin-option:: --traceback
  ```

  To link, use `:option:`command_name --traceback`` (or omit `command_name`
  for the options shared by all commands like `--verbosity`).
* Links to Trac tickets (typically reserved for patch release notes):
  ```rst
  :ticket:`12345`
  ```

Django’s documentation uses a custom `console` directive for documenting
command-line examples involving `django-admin`, `manage.py`, `python`,
etc.). In the HTML documentation, it renders a two-tab UI, with one tab showing
a Unix-style command prompt and a second tab showing a Windows prompt.

For example, you can replace this fragment:

```rst
use this command:

.. code-block:: console

    $ python manage.py shell
```

with this one:

```rst
use this command:

.. console::

    $ python manage.py shell
```

Notice two things:

* You usually will replace occurrences of the `.. code-block:: console`
  directive.
* You don’t need to change the actual content of the code example. You still
  write it assuming a Unix-y environment (i.e. a `'$'` prompt symbol,
  `'/'` as filesystem path components separator, etc.)

The example above will render a code example block with two tabs. The first
one will show:

```console
$ python manage.py shell
```

(No changes from what `.. code-block:: console` would have rendered).

The second one will show:

```doscon
...\> py manage.py shell
```

<a id="documenting-new-features"></a>

## Documenting new features

Our policy for new features is:

> All documentation of new features should be written in a way that
> clearly designates the features that are only available in the Django
> development version. Assume documentation readers are using the latest
> release, not the development version.

Our preferred way for marking new features is by prefacing the features’
documentation with: “`.. versionadded:: X.Y`”, followed by a mandatory
blank line and an optional description (indented).

General improvements or other changes to the APIs that should be emphasized
should use the “`.. versionchanged:: X.Y`” directive (with the same format
as the `versionadded` mentioned above.

These `versionadded` and `versionchanged` blocks should be “self-contained.”
In other words, since we only keep these annotations around for two releases,
it’s nice to be able to remove the annotation and its contents without having
to reflow, reindent, or edit the surrounding text. For example, instead of
putting the entire description of a new or changed feature in a block, do
something like this:

```rst
.. class:: Author(first_name, last_name, middle_name=None)

    A person who writes books.

    ``first_name`` is ...

    ...

    ``middle_name`` is ...

    .. versionchanged:: A.B

        The ``middle_name`` argument was added.
```

Put the changed annotation notes at the bottom of a section, not the top.

Also, avoid referring to a specific version of Django outside a
`versionadded` or `versionchanged` block. Even inside a block, it’s often
redundant to do so as these annotations render as “New in Django A.B:” and
“Changed in Django A.B”, respectively.

If a function, attribute, etc. is added, it’s also okay to use a
`versionadded` annotation like this:

```rst
.. attribute:: Author.middle_name

    .. versionadded:: A.B

    An author's middle name.
```

We can remove the `.. versionadded:: A.B` annotation without any indentation
changes when the time comes.

## Minimizing images

Optimize image compression where possible. For PNG files, use OptiPNG and
AdvanceCOMP’s `advpng`:

```console
$ cd docs
$ optipng -o7 -zm1-9 -i0 -strip all `find . -type f -not -path "./_build/*" -name "*.png"`
$ advpng -z4 `find . -type f -not -path "./_build/*" -name "*.png"`
```

This is based on OptiPNG version 0.7.5. Older versions may complain about the
`-strip all` option being lossy.

## An example

For a quick example of how it all fits together, consider this hypothetical
example:

* First, the `ref/settings.txt` document could have an overall layout
  like this:
  ```rst
  ========
  Settings
  ========

  ...

  .. _available-settings:

  Available settings
  ==================

  ...

  .. _deprecated-settings:

  Deprecated settings
  ===================

  ...
  ```
* Next, the `topics/settings.txt` document could contain something like
  this:
  ```rst
  You can access a :ref:`listing of all available settings
  <available-settings>`. For a list of deprecated settings see
  :ref:`deprecated-settings`.

  You can find both in the :doc:`settings reference document
  </ref/settings>`.
  ```

  We use the Sphinx [`doc`](https://www.sphinx-doc.org/en/master/usage/referencing.html#role-doc) cross-reference element when we want to
  link to another document as a whole and the [`ref`](https://www.sphinx-doc.org/en/master/usage/referencing.html#role-ref) element when
  we want to link to an arbitrary location in a document.
* Next, notice how the settings are annotated:
  ```rst
  .. setting:: ADMINS

  ADMINS
  ======

  Default: ``[]`` (Empty list)

  A list of all the people who get code error notifications. When
  ``DEBUG=False`` and a view raises an exception, Django will email these people
  with the full exception information. Each member of the list should be a tuple
  of (Full name, email address). Example::

      [("John", "john@example.com"), ("Mary", "mary@example.com")]

  Note that Django will email *all* of these people whenever an error happens.
  See :doc:`/howto/error-reporting` for more information.
  ```

  This marks up the following header as the “canonical” target for the
  setting `ADMINS`. This means any time I talk about `ADMINS`,
  I can reference it using `:setting:`ADMINS``.

That’s basically how everything fits together.

## Translating documentation

See [Localizing the Django documentation](localizing.md#translating-documentation) if
you’d like to help translate the documentation into another language.

<a id="django-admin-manpage"></a>

## `django-admin` man page

Sphinx can generate a manual page for the
[django-admin](../../ref/django-admin.md) command. This is configured in
`docs/conf.py`. Unlike other documentation output, this man page should be
included in the Django repository and the releases as
`docs/man/django-admin.1`. There isn’t a need to update this file when
updating the documentation, as it’s updated once as part of the release process.

To generate an updated version of the man page, run `make man` in the
`docs` directory. The new man page will be written in
`docs/_build/man/django-admin.1`.
