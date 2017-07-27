Outgoing Mail in Nice
^^^^^^^^^^^^^^^^^^^^^

Set Default Sender
------------------

The :term:`application property` ``email.default.from`` defines the global fallback sender address. It is used for mails
whose sender domain isn't allowed, see next section, or if no sender address is available.

.. code::

    email.default.from=noreply@tocco.ch


Allow Domains in Outgoing Mails
-------------------------------

Domains are allowed using the ``email.allowedFromDomainsRegex`` :term:`application property`. The value is a regular
expression matching the domains that are allowed as ``From`` in emails.

Mail send from a domain not covered by the given regular expression are rewritten to the mail address specified on the
business unit, if any, and ``email.default.from`` otherwise.

.. code::

    email.allowedFromDomainsRegex=tocco.ch|tocco.net

.. warning::

    Before you allow a domain make sure :doc:`SPF, DKIM and DMARC <dns_entries>` are set up.


Rewrite Recipient Addresses
---------------------------

The ``recipientrewrite.mappings`` :term:`application property` can be used to rewrite recipient addresses.

The syntax is very simple:

.. code::

    recipientrewrite.mappings=ORIGINAL -> REDIRECT1[, REDIRECT2]...[; ORIGINAL -> REDIRECT1[, REDIRECT2]...]...

==============  =====================================================================================================
ORIGINAL        Recipient address before rewriting. ``To``, ``Cc`` and ``Bcc`` addresses rewritten.
REDIRECTED1..   Rewrite rule for the address. You can specify multiple ``REDIRECT`` rules to have the mail redirected
                to multiple addresses at once.
==============  =====================================================================================================


Rewrite All Recipients
``````````````````````

A single asterisk (``*``) can be used to redirect all mails.

.. code::

    recipientrewrite.mappings=* -> fallback@tocco.ch


Rewrite One Domain
``````````````````

    Rewrite ``example.com`` to ``example.net``

    recipientrewrite.mappings=(.*?)@example\.com -> $1@example.net

.. note::

    ``$1`` matches the first group (=content of first ``( … )``) in the regular expression.


Rewrite to Multiple Recipients
``````````````````````````````

Rewrite ``xxx@tocco.ch`` to ``xxx-mail1@tocco.ch`` and ``xxx-mail2@tocco.ch``.

.. code::

    recipientrewrite.mappings=(.*?)@tocco.ch -> $1-mail1@tocco.ch, $1-mail2@tocco.ch;

.. note::

    The name of the recipient is not preserved if multiple recipients are used.


Combine Multiple Rules
``````````````````````

Redirect mails for ``example.com`` to ``example.net`` and redirect everything else to ``fallback@tocco.ch``.

.. code::

    recipientrewrite.mappings=(.*?)@example\.com -> $1@example.net; * -> fallback@tocco.ch

.. warning::

    Because the first matching pattern, from left to right, wins make sure the wildcard pattern (``*``) is at the
    very right.


Use a Custom Mail Server
------------------------

In case a customer wishes to use his own SMTP server the :term:`application property` ``email.hostname`` can be used
to set a custom mail server. (Our mail server is preconfigured.)

.. code::

    email.hostname=relay.cyberlink.ch
