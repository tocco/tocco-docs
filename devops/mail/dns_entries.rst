DNS Records for Outgoing Mails
==============================

Create SPF Record
-----------------

As shown below, ``include:spf.tocco.ch`` needs to be added to the SPF record. The record shown below may need to be
extended depending on whether another relay is used to send mails from the same domain. Also, ``?all`` is
generally safe but the domain owner may wish to use ``~all`` or ``-all`` to avoid that anyone else can send mails using
her domain.

.. code::

    @ IN TXT "v=spf1 … include:spf.tocco.ch ?all"

.. warning::

    ``~all`` and ``-all`` may lead to mails being rejected or considered spam if the policy is incorrect. Be careful!

Wikipedia has comprehensive article on `SPF`_ if more information is needed.

.. figure:: nine_spf.png
    :scale: 60%

    Sample SPF record in Nine web interface

.. _SPF: https://en.wikipedia.org/wiki/Sender_Policy_Framework


Create DKIM Record
------------------

A ``TXT`` entry for the name ``default._domainkey`` needs to be created.

.. code::

    default._domainkey IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtcJL5NHfaftQTFV9BemWPckwBj3Npls3ghFeh8e9RUFSpztQSMYeYVxYVJA7Km8QRX3zt3u3QgbIzp1rEjouHh03K0OsoKtQdmlBneg798peHI/MMwMrOVa8HFMyHW9JhhHiLdYNar9H77Ob1ourB6cAmTWFlaFQcFMF+o05Fhy5NCSVnsy/EWBHhLEII0d3iCMQJe/O19375x YVoDF494B1r323x4fNrHuTQcnxORaSSppXsYmCJ+SNoG+fIuVHYpxq2RCk/p9kuB0pNZl+wW7p2sdeknaDo5CYiQt/Wy4nHDiobq6SLuZ9pOpC652OodFuvIYI10npE/jbRpTZaQIDAQAB"

See wikipedia entry on `DKIM`_ for more details.

.. _DKIM: https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail

.. hint::

   In case the selector name ``default`` is already in use, a different selector can be configured for a domain. See
   `TOCO-23 <https://control.vshn.net/tickets/TOCO-23>`_ for details.

.. figure:: nine_dkim.png
    :scale: 60%

    Sample DKIM record in Nine web interface


Create DMARC Record
-------------------

A ``TXT`` entry for the name ``_dmarc`` needs to be created. Reports about mails rejected or classified as spam are sent
the the mail address specified in the record.

The subdomain policy, ``sp``, may need to be adjusted if the domain owner wishes to send mails from subdomains. The
policy ``p=none`` is generally safe but it can also be set to ``quarantine`` or ``reject`` if a stricter policy is
desired.

.. code::

    _dmarc IN TXT "v=DMARC1;p=none;sp=quarantine;pct=100;rua=mailto:dmarcreports@example.com"

.. warning::

    Be careful with ``quarantine`` and ``reject``. If SPF or DKIM records fail to validate the mail is going to be moved
    to spam or reject respectively.

Wikipedia has some more details on `DMARC`_.

.. _DMARC: https://en.wikipedia.org/wiki/DMARC

.. figure:: nine_dmarc.png
    :scale: 60%

    Sample DMARC record in Nine web interface


Verify Correctness
------------------

There are several online services that test the records for you. `Mail Tester`_ is one of the simpler ones and it covers
all the DNS entries listed above.

.. _Mail Tester: https://www.mail-tester.com/
