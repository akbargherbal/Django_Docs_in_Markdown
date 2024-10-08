# Localizing Django

Various parts of Django, such as the admin site and validation error messages,
are internationalized. This means they display differently depending on each
user’s language or country. For this, Django uses the same internationalization
and localization infrastructure available to Django applications, described in
the [i18n documentation](../../topics/i18n/index.md).

## Translations

Translations are contributed by Django users worldwide. The translation work is
coordinated at [Transifex](https://www.transifex.com/).

If you find an incorrect translation or want to discuss specific translations,
go to the [Django project page](https://app.transifex.com/django/django/). If you would like to help out with
translating or adding a language that isn’t yet translated, here’s what to do:

* Introduce yourself on the [Django internationalization forum](https://forum.djangoproject.com/c/internals/i18n/14).
* Make sure you read the notes about [Specialties of Django translation](../../topics/i18n/translation.md#specialties-of-django-i18n).
* Sign up at [Transifex](https://www.transifex.com/) and visit the [Django project page](https://app.transifex.com/django/django/).
* On the [Django project page](https://app.transifex.com/django/django/), choose the language you want to work on,
  **or** – in case the language doesn’t exist yet –
  request a new language team by clicking on the “Request language” link
  and selecting the appropriate language.
* Then, click the “Join this Team” button to become a member of this team.
  Every team has at least one coordinator who is responsible to review
  your membership request. You can also contact the team coordinator to clarify
  procedural problems and handle the actual translation process.
* Once you are a member of a team choose the translation resource you
  want to update on the team page. For example, the “core” resource refers
  to the translation catalog that contains all non-contrib translations.
  Each of the contrib apps also has a resource (prefixed with “contrib”).

  #### NOTE
  For more information about how to use Transifex, read the
  [Transifex User Guide](https://help.transifex.com/).

Translations from Transifex are only integrated into the Django repository at
the time of a new [feature release](../release-process.md#term-Feature-release). We try to update
them a second time during one of the following [patch release](../release-process.md#term-Patch-release)s, but that depends on the translation manager’s availability.
So don’t miss the string freeze period (between the release candidate and the
feature release) to take the opportunity to complete and fix the translations
for your language!

## Formats

You can also review `conf/locale/<locale>/formats.py`. This file describes
the date, time and numbers formatting particularities of your locale. See
[Format localization](../../topics/i18n/formatting.md) for details.

The format files aren’t managed by the use of Transifex. To change them, you
must:

* [Create a pull request](writing-code/submitting-patches.md) against the
  Django Git `main` branch, as for any code change.
* Open a ticket in Django’s ticket system, set its `Component` field to
  `Translations`, set the “has patch” flag, and include the link to the pull
  request.

<a id="translating-documentation"></a>

## Documentation

There is also an opportunity to translate the documentation, though this is a
huge undertaking to complete entirely (you have been warned!). We use the same
[Transifex tool](https://app.transifex.com/django/django-docs/). The
translations will appear at `https://docs.djangoproject.com/<language_code>/`
when at least the `docs/intro/*` files are fully translated in your language.

Once translations are published, updated versions from Transifex will be
irregularly ported to the [django/django-docs-translations](https://github.com/django/django-docs-translations) repository and to the
documentation website. Only translations for the latest stable Django release
are updated.
