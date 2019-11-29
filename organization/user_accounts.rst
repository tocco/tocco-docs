User Accounts
=============

Who get what Account?
---------------------

=========== ================== ===== ============
 Where       Role / Group       Dev   Ops/Admins
=========== ================== ===== ============
 Gerrit      \-                  x
 Gerrit      *Administrators*             x
 Teamcity    *developers*        x
 Teamcity    *admins*                     x
=========== ================== ===== ============

Gerrit
------

Create User
^^^^^^^^^^^

**${USERNAME}** is the username (e.g. *jdoe* for Jane Doe)

#. Generate password::

       pwgen -s 20 1

#. Create user::

       ssh -t tadm@git.tocco.ch sudo htpasswd /etc/nginx/.htpasswd ${USERNAME}

#. If admin access should be granted (see table above), go to go to the group
   `Administrators <https://git.tocco.ch/admin/groups/1,members>`__ and add the
   user as a member.


Deactivate User
^^^^^^^^^^^^^^^

.. code::

    ssh -p 29418 ${OWN_USERNAME}@git.tocco.ch "gerrit set-account --inactive ${USERNAME}"


TeamCity
--------

Permissions are granted via the *admins* and *developers* groups. New permission
should generally be granted to those groups rather than individual users.

Create User
^^^^^^^^^^^

#. Go to `Users <https://tc.tocco.ch/admin/admin.html?item=users>`__.
#. *Create User Account*
#. Enter *Username* (e.g. *jdoe* for Jane Doe)
#. Enter full name as *Name*
#. Enter *Email address* (e.g. jdoe@tocco.ch)
#. Generate a password (``pwgen -s 20 1``)
#. Select newly created user (click on username)
#. Select *Groups* tab.
#. Add user to group *admins* or *developers* according to the table
   above.

Remove User
^^^^^^^^^^^

#. Go to `Users <https://tc.tocco.ch/admin/admin.html?item=users>`__.
#. Remove user
