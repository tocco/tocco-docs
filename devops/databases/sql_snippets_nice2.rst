SQL Snippets (Nice)
===================

Assign All Non-Guest Roles to User 'tocco'
------------------------------------------

.. code:: sql

    INSERT INTO nice_login_role (
      _nice_version,
      _nice_create_timestamp,
      _nice_update_timestamp,
      _nice_create_user,
      _nice_update_user,
      fk_principal,
      fk_role,
      fk_business_unit,
      manual
    ) SELECT
      1,
      now(),
      now(),
      'tocco',
      'tocco',
      p.pk,
      r.pk,
      u.pk,
      true
    FROM
      nice_role AS r LEFT OUTER JOIN nice_role_type AS rt ON r.fk_role_type = rt.pk,
      nice_principal as p,
      nice_business_unit as u
    WHERE p.username = 'tocco' AND rt.unique_id <> 'guest'
    ON CONFLICT DO NOTHING;

Unblock User 'tocco'
--------------------

.. code:: sql

    UPDATE nice_principal
    SET
        fail_login_attempts = 0,
        fk_principal_status = (SELECT pk FROM nice_principal_status WHERE unique_id = 'active')
    WHERE username = 'tocco';

Set Password for User Tocco
---------------------------

Set password to ``NEW_PASSWORD``.

.. code:: sql

    UPDATE nice_principal
    SET
        password = md5('NEW_PASSWORD'), -- upgraded to PBKDF2 on next login
    WHERE username = 'tocco';
