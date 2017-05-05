Dump Database
=============

.. code:: bash

   pg_dump -U postgres -h db01master -Fc -f ~/_to_delete/nice2_${CUSTOMER}_$(date +"%Y_%m_%d").psql ${DATABASE};