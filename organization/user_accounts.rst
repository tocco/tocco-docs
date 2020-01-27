User Accounts
=============

Who get what Account?
---------------------

=================== ======================== ===== ============
 Where               Role / Group             Dev   Ops/Admins
=================== ======================== ===== ============
 Gerrit              \-                        x
 Gerrit              *Administrators*                   x
 Teamcity            *developers*              x
 Teamcity            *admins*                           x
 VSHN                \-                        x        x
 VSHN SSH            *tocco*                   x
 VSHN SSH            *toccoroot* and                    x
                     *tocco*
 SSH Server Access   *@user*                   x
 SSH Server Access   *@root*                            x
 GitHub              *Member*                  x
 GitHub              *Owner*                            x
 GitLab              *Maintainer*              x
 GitLab              *Owner*                            x
 Docker Hub          *Owner*                            x
 Read the Docs       *Maintainer*                       x
 Sonar               *sonar-administrators*             x
 Sonar               *sonar-users*             x
=================== ======================== ===== ============

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


.. _ssh-server-access-ansible:

SSH Server Access (Ansible)
---------------------------

Allow Access
------------


    Access can be granted via `roles/ssh-key-sync/files/ssh_keys`_ in the Ansible repository.

    Changes can be deployed via Ansible::

        cd ${ANSIBLE_GIT_ROOT}
        ansible-playbook -i inventory playbook.yml -t ssh-keys

    .. hint::

        Users with role ``@user`` have access as user *tocco* on some hosts. User with role ``@root`` have access as
        *tadm* and *tocco* on all hosts.


Revoke Access
-------------

    See *Allow Access* above and do the opposite.


VSHN
----

Create Account
^^^^^^^^^^^^^^

Create a ticket to have an account created. See :vshn:`TOCO-183` for an example.

Remove Account
^^^^^^^^^^^^^^

Create a ticket to have an account removed. See :vshn:`TOCO-192` for an example.


.. _vshn-ssh-access:

VSHN SSH
--------

Grant Access
^^^^^^^^^^^^

    Puppet configuration can be found in the `tocco_hieradata repository`_. Access is defined in
    the ``users`` section within the different config files (e.g. in ``database.yml`` for
    database servers and ``infrastructure/solr.yml`` for Solr servers).

    .. hint::

        Users that are part of the group ``toccoroot`` can use sudo to obtain root priviledges.


Revoke Access
^^^^^^^^^^^^^

    To remove an account, add an ``ensure: absent``.


GitHub
------

Add User to Organization
^^^^^^^^^^^^^^^^^^^^^^^^

Go to the `People page`_ and *Invite member*.

Remove User from Organization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Go to the `People page`_ and select *Convert to outside collaborator…*.


GitLab
------

Add User to Group
^^^^^^^^^^^^^^^^^

Go to the `Members page`_ and add the user.


Remove User from Group
^^^^^^^^^^^^^^^^^^^^^^

Go to the `Members page`_ and remove the user.


Docker Hub
----------

Add User to Organization
^^^^^^^^^^^^^^^^^^^^^^^^

Go to the `Docker Hub's Members page`_ and add the user.

Remove User from Organization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Go to the `Docker Hub's Members page`_ and remove the user.


Read the Docs
-------------

Grant Access
^^^^^^^^^^^^

Add user as maintainer on `Read the Docs' Users page`_.

Revoke Access
^^^^^^^^^^^^^

Remove user as maintainer on `Read the Docs' Users page`_.


Sonar
-----

Create Account
--------------

Add user on `Sonar's Users page`_.

Remove Account
--------------

Remove user on `Sonar's Users page`_.


.. _roles/ssh-key-sync/files/ssh_keys: https://git.tocco.ch/gitweb?p=ansible.git;a=blob;f=roles/ssh-key-sync/files/ssh_keys
.. _tocco_hieradata repository: https://git.vshn.net/tocco/tocco_hieradata/tree/master
.. _People page: https://github.com/orgs/tocco/people
.. _Members page: https://gitlab.com/groups/toccoag/-/group_members
.. _Docker Hub's Members page: https://hub.docker.com/orgs/toccoag
.. _Read the Docs' Users page: https://readthedocs.org/dashboard/tocco-docs/users/
.. _Sonar's Users page: https://sonar.tocco.ch/admin/users
