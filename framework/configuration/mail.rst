Mail
====

IMAP
----

IMAP can be used to fetch mails periodically from an inbox or any other folder.


The following configuration is required to setup IMAP::

    # How often to check for new mails (cron syntax, every two minutes in this example)
    nice2.optional.mailintegration.schedule=*/2 * * * *

    # Whether to actually fetch mails via IMAP
    nice2.optional.mailintegration.active=true

    # URL of the IMAP server
    nice2.optional.mailintegration.mailbox.default.url=imaps://test%40tocco.ch:SECRET-PASSWORD@mail.tocco.ch/INBOX

    # Whether to fetch mails from subfolders
    nice2.optional.mailintegration.mailbox.default.readRecursive=false

    # Whether to remove mails after fetching them
    nice2.optional.mailintegration.mailbox.default.removeOnFinish=true


The URL specified as ``nice2.optional.mailintegration.mailbox.default.url`` comprises these parts::

    <protocol>://<user>:<password>@<server>[:<port>]/<folder>

============ ===============================================================================
 <protocol>   Either ``imap`` for explicit TLS (port 143) or ``imaps`` for
              implicit TLS (port 993).
 <user>       Username. The *@* character needs to be percentage encoded as *%40*. So,
              *peter@tocco.ch* would becomes *peter%40tocco.ch*.
 <password>   Password of ${USER}
 <server>     Name of the IMAP server
 <port>       (optional) Specify a custom port to use. See ${PROTOCOL} for default ports.
 <folder>     Folder from which to fetch mails. The inbox is called *INBOX*. Folder
              separator is usually */*. Thus, *INBOX/tocco* selects the subfolder *tocco*
              of the *INBOX* folder.
============ ===============================================================================


POP3
----

.. warning::

    While pop3 support is implemented, it is not used actively and has not
    been tested. Thus, **do not expect pop3 support to just work**.

POP3 is an older alternative to IMAP. Use IMAP whenever possible.

The setup is the same as for IMAP described above except the url given as
``nice2.optional.mailintegration.mailbox.default.url`` differs slightly::

    <protocol>://<user>:<password>@<server>[:<port>]

============ ===============================================================================
 <protocol>   Either ``pop3`` for explicit TLS (port 110) or ``pop3s`` for
              implicit TLS (port 995).
 <...>        See IMAP above.
============ ===============================================================================
