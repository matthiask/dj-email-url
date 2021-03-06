============
dj-email-url
============

.. image:: https://badge.fury.io/py/dj-email-url.svg
    :target: http://badge.fury.io/py/dj-email-url

This utility is based on dj-database-url by Kenneth Reitz.

It allows to utilize the
`12factor <http://www.12factor.net/backing-services>`_ inspired
environments variable to configure the email backend in a Django application.

Usage
=====

Import the package in ``settings.py``:

.. code:: python

    import dj_email_url


Fetch your email configuration values. The default option is fetch them from
``EMAIL_URL`` environment variable:

.. code:: python

    email_config = dj_email_url.config()

Other option is parse an arbitrary email URL:

.. code:: python

    email_config = dj_email_url.parse('smtp://...')


Finally, it is **necessary** to assign values to settings:

.. code:: python

    EMAIL_FILE_PATH = email_config['EMAIL_FILE_PATH']
    EMAIL_HOST_USER = email_config['EMAIL_HOST_USER']
    EMAIL_HOST_PASSWORD = email_config['EMAIL_HOST_PASSWORD']
    EMAIL_HOST = email_config['EMAIL_HOST']
    EMAIL_PORT = email_config['EMAIL_PORT']
    EMAIL_BACKEND = email_config['EMAIL_BACKEND']
    EMAIL_USE_TLS = email_config['EMAIL_USE_TLS']
    EMAIL_USE_SSL = email_config['EMAIL_USE_SSL']

Alternatively, it is possible to use this less explicit shortcut:

.. code:: python

    vars().update(email_config)

Common EMAIL_URL values
-----------------------

====================================================================== ======================================================
EMAIL_URL                                                              Description
====================================================================== ======================================================
``console:``                                                           Email printed on screen (development)
``smtp:``                                                              Email sent using a mail transfer agent at localhost
``submission://sendgrid_username:sendgrid_password@smtp.sendgrid.com`` Email sent using SendGrid_ SMTP on port 587 (STARTTLS)
====================================================================== ======================================================

.. _SendGrid: https://sendgrid.com/docs/Integrate/Frameworks/django.html

Supported backends
==================

Currently, `dj-email-url` supports:

- SMTP backend
  (``smtp`` for port 25, ``submission`` or ``submit`` for port 587),

- console backend (``console``),

- file backend (``file``),

- in-memory backend (``memory``),

- and dummy backend (``dummy``).

SMTP backend
------------

The `SMTP backend`__ is selected when the scheme in the URL if one these:

__ https://docs.djangoproject.com/en/dev/topics/email/#smtp-backend

============================ ============ =========================
Value                        Default port Comment
============================ ============ =========================
``smtp``                     25           Local mail transfer agent
``submission`` or ``submit`` 587          SMTP with STARTTLS
============================ ============ =========================


*Changed in version 0.1:* The use of ``smtps`` is now discouraged__
The value ``smtps`` was used to indicate to use TLS connections,
that is to set ``EMAIL_USE_TLS`` to ``True``.
Now is recommended to use ``submission`` or ``submit``
(see `service name for port numbers`_ or `Uniform Resource Identifier Schemes`_ at IANA).

__ SMTPS_

.. _SMTPS: https://en.wikipedia.org/wiki/SMTPS

.. _service name for port numbers: https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=587

.. _Uniform Resource Identifier Schemes: https://www.iana.org/assignments/uri-schemes/uri-schemes.xhtml

On the most popular mail configuration option is
to use a **third party SMTP server to relay emails**.

.. code:: pycon

    >>> url = 'submission://user@example.com:pass@smtp.example.com'
    >>> url = dj_email_url.parse(url)
    >>> assert url['EMAIL_PORT'] == 587
    >>> assert url['EMAIL_USE_SSL'] is False
    >>> assert url['EMAIL_USE_TLS'] is True

Other common option is to use a **local mail transfer agent** Postfix or Exim.
In this case it as easy as:

.. code:: pycon

    >>> url = 'smtp://'
    >>> url = dj_email_url.parse(url)
    >>> assert url['EMAIL_HOST'] == 'localhost'
    >>> assert url['EMAIL_PORT'] == 25
    >>> assert url['EMAIL_USE_SSL'] is False
    >>> assert url['EMAIL_USE_TLS'] is False

It is also possible to configure **SMTP-over-SSL** (usually on 465).
This configuration is not generally recommended but might be needed for legacy systems.
To apply use this configuration specify SSL using a `ssl=True` as a query parameter
and indicate the port explicitly:

.. code:: pycon

    >>> url = 'smtp://user@domain.com:pass@smtp.example.com:465/?ssl=True'
    >>> url = dj_email_url.parse(url)
    >>> assert url['EMAIL_PORT'] == 465
    >>> assert url['EMAIL_USE_SSL'] is True
    >>> assert url['EMAIL_USE_TLS'] is False

File backend
------------

The file backend is the only one which needs a path. The url path is store
in ``EMAIL_FILE_PATH`` key.

Change Log
==========

Unreleased
----------

0.1.0_ - 2018-03-24
-------------------

.. _0.1.0: https://pypi.python.org/pypi/dj-email-url/0.1.0

- Added new schemes ``submission`` and ``submit``
  to select SMTP backend on port 587 with STARTTLS.
  Thanks to @LEW21 to suggest to include new `submit` URI.

- Discouraged the use of scheme ``smtps`` and add a user warning.
  Thanks to @LEW21 to alert about this confusing usage.

- Expand which values are considered as truthy on a query string param. Now,
  `1`, `on`, `true`, and `yes`, as a single character or in all case variants
  (lower, upper and title case) are considered as `True`.

0.0.10_ - 2016-10-14
--------------------

- Post release version to fix release date in change log.

0.0.9_ - 2016-10-14
-------------------

- Fix case when user sets ssl=False in its url (thanks bogdal)

0.0.8_ - 2016-06-07
-------------------

- Allow universal wheel

0.0.7_ - 2016-05-31
-------------------

- Add EMAIL_USE_SSL setting to docs and set a default value (thanks iraycd).
- Add coverage (thanks iraycd).

0.0.6_ - 2016-04-18
-------------------

- Fix error parsing URL without credentials (thanks martinmaillard).

0.0.5_ - 2016-04-17
-------------------

- Allow URL encoded credentials (thanks kane-c).

0.0.4_ - 2015-03-05
-------------------

- Fix README.

0.0.3_ - 2015-03-05
-------------------

- Add change log.

- Add `ssl=` option as a query parameter for SMTP backend.

- Add Travis continuous integration.

0.0.2_ - 2014-03-12
-------------------

- Add Python 3 support.

0.0.1_ - 2013-02-12
-------------------

- Initial version.

.. _0.0.1: https://pypi.python.org/pypi/dj-email-url/0.0.1
.. _0.0.2: https://pypi.python.org/pypi/dj-email-url/0.0.2
.. _0.0.3: https://pypi.python.org/pypi/dj-email-url/0.0.3
.. _0.0.4: https://pypi.python.org/pypi/dj-email-url/0.0.4
.. _0.0.5: https://pypi.python.org/pypi/dj-email-url/0.0.5
.. _0.0.6: https://pypi.python.org/pypi/dj-email-url/0.0.6
.. _0.0.7: https://pypi.python.org/pypi/dj-email-url/0.0.7
.. _0.0.8: https://pypi.python.org/pypi/dj-email-url/0.0.8
.. _0.0.9: https://pypi.python.org/pypi/dj-email-url/0.0.9
.. _0.0.10: https://pypi.python.org/pypi/dj-email-url/0.0.10

CI status
=========

Development (master):

.. image:: https://travis-ci.org/migonzalvar/dj-email-url.svg?branch=master
  :target: http://travis-ci.org/migonzalvar/dj-email-url

.. image:: https://codecov.io/gh/migonzalvar/dj-email-url/branch/master/graph/badge.svg
  :target: https://codecov.io/gh/migonzalvar/dj-email-url
