Mail
====

Incorrect SMTP Server
---------------------

Error
^^^^^

.. code::

    MessagingException: Could not connect to SMTP host: …


Solution
^^^^^^^^

Ensure the correct smtp host is configured (in application.local.properties):

.. code::

    email.hostname=mxout1.tocco.cust.vshn.net

.. note::

    The relay shown is the relay used for all installations running on our infrastructure. It may differ for customers
    hosting Nice themselves.


Full Error
^^^^^^^^^^

.. code::

    Caused by: javax.mail.MessagingException: Could not connect to SMTP host: relay.cyberlink.ch, port: 25, response: 554
            at com.sun.mail.smtp.SMTPTransport.openServer(SMTPTransport.java:2088)
            at com.sun.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:699)
            at javax.mail.Service.connect(Service.java:366)
            at javax.mail.Service.connect(Service.java:246)
            at javax.mail.Service.connect(Service.java:195)
            at javax.mail.Transport.send0(Transport.java:254)
            at javax.mail.Transport.send(Transport.java:124)
            at ch.tocco.nice2.messaging.mail.impl.MailImpl.send(MailImpl.java:437)
            ... 21 common frames omitted
