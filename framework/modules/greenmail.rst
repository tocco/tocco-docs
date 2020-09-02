Greenmail
=========

Additionally to the mail module their is the Greenmail mail module. It starts a test mail server and catches all sending e-mails.

.. list-table::

   * - SMTP
     - localhost:3025
   * - POP
     - localhost:3110
   * - IMAP
     - localhost:3143
   * - User
     - greenmail
   * - Password
     - greenmail

The user is a catch-all user, i.e. all mails that are sent end up in his mailbox, so you can pick them up via POP3 or IMAP.
You can also configure more users in the ``hivemodule.xml`` file. JavaMail is automatically configured correctly
if this module is present at startup and the "normal" email configuration is ignored.

To disable such a test mail server, just set the corresponding option to off,
e.g. ``nice2.messaging.greenmail.imapSetup=off``, so that the IMAP server will not be started.