Restore Database
================

.. code:: bash

   pg_restore -j 4 -h postgres -U postgres --role ${DB_USER} --no-role --no-acl -d ${DB_NAME} ${DUMP_FILE_PATH}
