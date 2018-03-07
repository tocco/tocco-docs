Postgres Remote Access
======================

Using SSH Port Forwarding
-------------------------

.. note::

    This requires ssh access to the DB server.

.. parsed-literal::

    ssh ${USERNAME}@db1.tocco.cust.vshn.net -L 5432:localhost:5432 -N

Now you should be able to connect to the DB server on **locahost:5432**.


Direct Access
-------------

.. note::

    Direct access is only possible from whitelisted addresses.

.. important::

    Postgres doesn't enforce SSL by default, you **must** enable it. Take a look at `libpq - SSL Support`_ for more
    details.


Using PSQL
``````````

Download the **${CERT}** for :download:`db1.tocco.cust.vshn.net </_static/download/db1.tocco.cust.vshn.net.pem>` /
:download:`db2.tocco.cust.vshn.net </_static/download/db2.tocco.cust.vshn.net.pem>` first.

.. parsed-literal::

    psql 'postgresql://**${USER}**\ @db1.tocco.cust.vshn.net/**${DB_NAME}**?sslmode=verify-full&sslrootcert=\ **${CERT}**'



Using Python
````````````

Download the **CERT** for :download:`db1.tocco.cust.vshn.net </_static/download/db1.tocco.cust.vshn.net.pem>` /
:download:`db2.tocco.cust.vshn.net </_static/download/db2.tocco.cust.vshn.net.pem>` first.

.. code-block:: python3

    import psycopg2

    conn = psycopg2.connect(
        host = "db1.tocco.cust.vshn.net",
        database = DB_NAME,
        user = USER,
        password = PASSWORD,
        sslmode = "verify-full",
        sslrootcert = CERT
    )


Other Means of Accessing Postgres
`````````````````````````````````

There are many more libraries and tools that allow you to access a Postgres DB server. But be aware that Postgres doesn't
enable SSL verification by default, **you must make sure SSL certificates are verified!**  Take a look at
`libpq - SSL Support`_, most tools and libraries based on libpg. Thus, most of them use the same SSL settings.

Certificates: :download:`db1.tocco.cust.vshn.net </_static/download/db1.tocco.cust.vshn.net.pem>` /
:download:`db2.tocco.cust.vshn.net </_static/download/db2.tocco.cust.vshn.net.pem>`


.. _libpq - SSL Support: https://www.postgresql.org/docs/current/static/libpq-ssl.html
