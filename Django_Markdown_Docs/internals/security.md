# Django’s security policies

Django’s development team is strongly committed to responsible
reporting and disclosure of security-related issues. As such, we’ve
adopted and follow a set of policies which conform to that ideal and
are geared toward allowing us to deliver timely security updates to
the official distribution of Django, as well as to third-party
distributions.

<a id="reporting-security-issues"></a>

## Reporting security issues

**Short version: please report security issues by emailing
security@djangoproject.com**.

Most normal bugs in Django are reported to [our public Trac instance](https://code.djangoproject.com/query), but
due to the sensitive nature of security issues, we ask that they **not** be
publicly reported in this fashion.

Instead, if you believe you’ve found something in Django which has security
implications, please send a description of the issue via email to
`security@djangoproject.com`. Mail sent to that address reaches the [security
team](https://www.djangoproject.com/foundation/teams/#security-team).

Once you’ve submitted an issue via email, you should receive an acknowledgment
from a member of the security team within 48 hours, and depending on the
action to be taken, you may receive further followup emails.

<a id="security-support"></a>

## Supported versions

At any given time, the Django team provides official security support
for several versions of Django:

* The [main development branch](https://github.com/django/django/), hosted on GitHub, which will become the
  next major release of Django, receives security support. Security issues that
  only affect the main development branch and not any stable released versions
  are fixed in public without going through the [disclosure process](#security-disclosure).
* The two most recent Django release series receive security
  support. For example, during the development cycle leading to the
  release of Django 1.5, support will be provided for Django 1.4 and
  Django 1.3. Upon the release of Django 1.5, Django 1.3’s security
  support will end.
* [Long-term support release](release-process.md#term-Long-term-support-release)s will receive security updates for a
  specified period.

When new releases are issued for security reasons, the accompanying
notice will include a list of affected versions. This list is
comprised solely of *supported* versions of Django: older versions may
also be affected, but we do not investigate to determine that, and
will not issue patches or new releases for those versions.

<a id="security-disclosure"></a>

## How Django discloses security issues

Our process for taking a security issue from private discussion to
public disclosure involves multiple steps.

Approximately one week before public disclosure, we send two notifications:

First, we notify [django-announce](mailing-lists.md#django-announce-mailing-list) of the date and approximate time of the
upcoming security release, as well as the severity of the issues. This is to
aid organizations that need to ensure they have staff available to handle
triaging our announcement and upgrade Django as needed. Severity levels are:

* **High**
  * Remote code execution
  * SQL injection
* **Moderate**
  * Cross site scripting (XSS)
  * Cross site request forgery (CSRF)
  * Denial-of-service attacks
  * Broken authentication
* **Low**
  * Sensitive data exposure
  * Broken session management
  * Unvalidated redirects/forwards
  * Issues requiring an uncommon configuration option

Second, we notify a list of [people and organizations](#security-notifications), primarily composed of operating-system vendors and
other distributors of Django. This email is signed with the PGP key of someone
from [Django’s release team](https://www.djangoproject.com/foundation/teams/#releasers-team) and consists of:

* A full description of the issue and the affected versions of Django.
* The steps we will be taking to remedy the issue.
* The patch(es), if any, that will be applied to Django.
* The date on which the Django team will apply these patches, issue
  new releases and publicly disclose the issue.

On the day of disclosure, we will take the following steps:

1. Apply the relevant patch(es) to Django’s codebase.
2. Issue the relevant release(s), by placing new packages on the [Python
   Package Index](https://pypi.org/project/Django/) and on the [djangoproject.com website](https://www.djangoproject.com/download/), and tagging the new release(s)
   in Django’s git repository.
3. Post a public entry on [the official Django development blog](https://www.djangoproject.com/weblog/),
   describing the issue and its resolution in detail, pointing to the
   relevant patches and new releases, and crediting the reporter of
   the issue (if the reporter wishes to be publicly identified).
4. Post a notice to the [django-announce](mailing-lists.md#django-announce-mailing-list) and [oss-security@lists.openwall.com](mailto:oss-security@lists.openwall.com)
   mailing lists that links to the blog post.

If a reported issue is believed to be particularly time-sensitive –
due to a known exploit in the wild, for example – the time between
advance notification and public disclosure may be shortened
considerably.

Additionally, if we have reason to believe that an issue reported to
us affects other frameworks or tools in the Python/web ecosystem, we
may privately contact and discuss those issues with the appropriate
maintainers, and coordinate our own disclosure and resolution with
theirs.

The Django team also maintains an [archive of security issues
disclosed in Django](../releases/security.md).

<a id="security-notifications"></a>

## Who receives advance notification

The full list of people and organizations who receive advance
notification of security issues is not and will not be made public.

We also aim to keep this list as small as effectively possible, in
order to better manage the flow of confidential information prior to
disclosure. As such, our notification list is *not* simply a list of
users of Django, and being a user of Django is not sufficient reason
to be placed on the notification list.

In broad terms, recipients of security notifications fall into three
groups:

1. Operating-system vendors and other distributors of Django who
   provide a suitably-generic (i.e., *not* an individual’s personal
   email address) contact address for reporting issues with their
   Django package, or for general security reporting. In either case,
   such addresses **must not** forward to public mailing lists or bug
   trackers. Addresses which forward to the private email of an
   individual maintainer or security-response contact are acceptable,
   although private security trackers or security-response groups are
   strongly preferred.
2. On a case-by-case basis, individual package maintainers who have
   demonstrated a commitment to responding to and responsibly acting
   on these notifications.
3. On a case-by-case basis, other entities who, in the judgment of the
   Django development team, need to be made aware of a pending
   security issue. Typically, membership in this group will consist of
   some of the largest and/or most likely to be severely impacted
   known users or distributors of Django, and will require a
   demonstrated ability to responsibly receive, keep confidential and
   act on these notifications.

## Requesting notifications

If you believe that you, or an organization you are authorized to
represent, fall into one of the groups listed above, you can ask to be
added to Django’s notification list by emailing
`security@djangoproject.com`. Please use the subject line “Security
notification request”.

Your request **must** include the following information:

* Your full, real name and the name of the organization you represent,
  if applicable, as well as your role within that organization.
* A detailed explanation of how you or your organization fit at least
  one set of criteria listed above.
* A detailed explanation of why you are requesting security notifications.
  Again, please keep in mind that this is *not* simply a list for users of
  Django, and the overwhelming majority of users should subscribe to
  [django-announce](mailing-lists.md#django-announce-mailing-list) to receive advanced notice of when a security release will
  happen, without the details of the issues, rather than request detailed
  notifications.
* The email address you would like to have added to our notification
  list.
* An explanation of who will be receiving/reviewing mail sent to that
  address, as well as information regarding any automated actions that
  will be taken (i.e., filing of a confidential issue in a bug
  tracker).
* For individuals, the ID of a public key associated with your address
  which can be used to verify email received from you and encrypt
  email sent to you, as needed.

Once submitted, your request will be considered by the Django
development team; you will receive a reply notifying you of the result
of your request within 30 days.

Please also bear in mind that for any individual or organization,
receiving security notifications is a privilege granted at the sole
discretion of the Django development team, and that this privilege can
be revoked at any time, with or without explanation.
