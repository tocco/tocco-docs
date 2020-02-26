####
Mail
####


Mails End Up in Spam
====================

Before starting the investigation, inquire this information and
write it down on the ticket:

* When was the issue noticed first?
* What sender address(es) are affected?
* What recipient address(es) are affected?
* How was the mail sent? (manually, automated mail during
  registration, etc.) (Include exact steps to reproduce.)
* Was a specific template used to send the mail? Does the
  issue occur when another or no template is used?
* Who sent the mail(s) (login(s))?
* Does the message end up in spam with a specific provider
  (e.g. Google, Bluewin, Gmx)?

Analyzing the issue:

* Create a Mail Tester report as described
  `here <https://www.tocco.ch/intranet/Infos-&-Events/Blogs#post&key=1657>`__.
* Attach a link **and** a full-page screenshot to the ticket. Before taking
  the screenshot, make sure all test results are shown (click all pluses).

Common issues:

* Incorrect or missing SPF, see `Create / Adjust SPF Record`_.
* Incorrect or missing DKIM, see `Create DKIM Record`_.


Sender Address Selection
========================

It is possible for a user to select an arbitrary sender address. However,
we are not necessarily allowed to send mails on behalf that domain. Thus,
we restrict what sender domains are allowed and if a domain is not allowed,
we rewrite the sender to a fallback address:

.. graphviz::

    digraph {
        label="Selecting a sender address."

        given_address [ label="selected/entered address" ]
        is_in_allowed_domains [ shape=diamond, label="Is sender in\nallowed domains?" ]
        has_fallback_on_bu [ shape=diamond, label="Is default address\ndefined on BU?" ]
        end [ shape=circle, label="" ]

        given_address -> is_in_allowed_domains
        is_in_allowed_domains -> end [ label="yes, use given address" ]
        is_in_allowed_domains -> has_fallback_on_bu [ label="no" ]
        has_fallback_on_bu -> end [ label="yes, use address from BU" ]
        has_fallback_on_bu -> end [ label="no, use global default address" ]
    }

Essentially, the entered sender address is only used if it's the configured list of allowed
domains. Otherwise, a fallback address is used.

TODO: How does BS 1st find out what domains are allowed?

(:doc:`technical documentation </devops/mail/outgoing_mail_in_nice>`)


Create DKIM Record
==================

1. Check if the DKIM entry exists::

       $ dig -t txt default._domainkey.$DOMAIN

   Alternatively, use `this web-based tool <https://dnslookup.org/default._domainkey.tocco.ch/TXT/#delegation>`__.

2. Check if the record is correct.

   The record must be::

       default._domainkey IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtcJL5NHfaftQTFV9BemWPckwBj3Npls3ghFeh8e9RUFSpztQSMYeYVxYVJA7Km8QRX3zt3u3QgbIzp1rEjouHh03K0OsoKtQdmlBneg798peHI/MMwMrOVa8HFMyHW9JhhHiLdYNar9H77Ob1ourB6cAmTWFlaFQcFMF+o05Fhy5NCSVnsy/EWBHhLEII0d3iCMQJe/O19375x YVoDF494B1r323x4fNrHuTQcnxORaSSppXsYmCJ+SNoG+fIuVHYpxq2RCk/p9kuB0pNZl+wW7p2sdeknaDo5CYiQt/Wy4nHDiobq6SLuZ9pOpC652OodFuvIYI10npE/jbRpTZaQIDAQAB"

   (The record is the same for all installations and domains.)

3. Create/DNS record

   First find out :ref:`dns-who-updates-record`

   If we manage DNS adjust or add the DNS record yourself. The record should
   look like this:

   .. figure:: /devops/mail/nine_dkim.png
      :scale: 60%

      Nine DNS configuration web interface.

   If we don't manage DNS, inform the customer that the following changes are required:

       | fqdn: ``default._domainkey.operations-the-rat.ch`` (**example!**)
       | type: ``TXT``
       | value: ``v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtcJL5NHfaftQTFV9BemWPckwBj3Npls3ghFeh8e9RUFSpztQSMYeYVxYVJA7Km8QRX3zt3u3QgbIzp1rEjouHh03K0OsoKtQdmlBneg798peHI/MMwMrOVa8HFMyHW9JhhHiLdYNar9H77Ob1ourB6cAmTWFlaFQcFMF+o05Fhy5NCSVnsy/EWBHhLEII0d3iCMQJe/O19375x YVoDF494B1r323x4fNrHuTQcnxORaSSppXsYmCJ+SNoG+fIuVHYpxq2RCk/p9kuB0pNZl+wW7p2sdeknaDo5CYiQt/Wy4nHDiobq6SLuZ9pOpC652OodFuvIYI10npE/jbRpTZaQIDAQAB``


Create / Adjust SPF Record
==========================

1. Check if a SPF entry exists:

   Command line::

       dig -t txt $DOMAIN

   Or use an online tool like the `SPF Record Testing Tools`_. 

   (SPF records are the ones starting with ``v=spf1``.)

2a. Extend an existing SPF record:

    In case a record already exists, only ``include:spf.tocco.ch`` needs
    to be added to it.

    For instance, given this pre-existing record::

       v=spf1 include:spf.protection.outlook.com -all

    the additional *include* should be added like this::

       v=spf1 include:spf.protection.outlook.com include:spf.tocco.ch -all

    Always include the *include* in between ``v=spf1`` and ``all``. Those need
    to stay at the beginning and end respectively.


2b. Create a new SPF record:

    When creating a new SPF record, care needs to be taken that third party mail
    server sending mails in the name of the domain are allowed as sender. Thus,
    **it is important to ask the customer if they send mails for a given domain
    using any other providers (e.g. do they send mails via Google or Office 365).**

    This is the basic record needed::

        v=spf1 include:spf.tocco.ch -all

    This only allows our mail server, if more mail servers are used for a domain,
    they need to be allowed additionally. Here an example::

        v=spf1 include:spf.tocco.ch include:_spf.protonmail.com include:spf.mailjet.com -all

    As shown in the above example, additional SPF record are best included using the ``include:``
    keyword.

3. Validate:

   Check the validity of the SPF record using the check *Is This SPF record
   valid - syntactically correct?* which can be found amongst the `SPF Record
   Testing Tools`_.

4. Create/DNS record

   First find out :ref:`dns-who-updates-record`

   If we manage DNS adjust or add the DNS record yourself. The record should
   look like this:

   .. figure:: /devops/mail/nine_spf.png
      :scale: 60%

      Nine DNS configuration web interface.

   If we don't manage DNS, inform the customer that the following changes are required:

       | fqdn: ``operations-the-rat.ch`` (**example!**)
       | type: ``TXT``
       | value: ``v=spf1 include:spf.tocco.ch -all`` (**example!**)

   **Tell the customer if the record needs to be updated or added.** Customers tend to add
   new record even if one exists already.


.. _SPF Record Testing Tools: https://www.kitterman.com/spf/validate.html
