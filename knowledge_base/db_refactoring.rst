DB Refactoring
==============

Incorrect Business Units
------------------------

Error
^^^^^

.. code::

    org.postgresql.util.PSQLException: ERROR: syntax error at or near "true"

Solution
^^^^^^^^

    The business units set in application properties must match the business units found on the db.

    Business units in DB: ``SELECT unique_id FROM nice_business_unit``

    Business units in application properties: Look for property named ``nice2.dbrefactoring.businessunits``.

Full Error Message
^^^^^^^^^^^^^^^^^^

.. code::

    Fragments
    ----------

    >> CreateBusinessUnitFragment: .......... FAILED
    >> SchemaUpgrade: .......... SKIPPED
    >> BinaryFkFragment: .......... SKIPPED
    >> AddForeignKeysIndex: .......... SKIPPED
    >> FixCountersFragment: .......... SKIPPED
    >> AddDmsFksFragment: .......... SKIPPED
    >> AddContentReferenceSourceFksFragment: .......... SKIPPED
    >> AddOrderColumnIndexFragment: .......... SKIPPED

    Error Message:
    --------------

    org.postgresql.util.PSQLException: ERROR: syntax error at or near "true"
      Position: 1612
        at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2284)
        at org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2003)
        at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:200)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:424)
        at org.postgresql.jdbc.PgPreparedStatement.executeWithFlags(PgPreparedStatement.java:161)
        at org.postgresql.jdbc.PgPreparedStatement.executeUpdate(PgPreparedStatement.java:133)
        at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)
        at com.zaxxer.hikari.pool.HikariProxyPreparedStatement.executeUpdate(HikariProxyPreparedStatement.java)
        at ch.tocco.nice2.dbrefactoring.impl.install.fragments.CreateBusinessUnitFragment.createBusinessUnit(CreateBusinessUnitFragment.java:312)
        at ch.tocco.nice2.dbrefactoring.impl.install.fragments.CreateBusinessUnitFragment.handleBusinessUnits(CreateBusinessUnitFragment.java:132)
        at ch.tocco.nice2.dbrefactoring.impl.install.fragments.CreateBusinessUnitFragment.install(CreateBusinessUnitFragment.java:95)
        at $ch.tocco.nice2.dbrefactoring.impl.install.fragments.CreateBusinessUnitFragment_15f25e2b90e.install(CreateBusinessUnitFragment_15f25e2b90e.java)
        at $InstallFragment_15f25e2b8c3.install($InstallFragment_15f25e2b8c3.java)
        at $InstallFragment_15f25e2b8c2.install($InstallFragment_15f25e2b8c2.java)
        at ch.tocco.nice2.dbrefactoring.impl.install.InstallationStepImpl.installStep(InstallationStepImpl.java:121)
        at $ch.tocco.nice2.dbrefactoring.impl.install.InstallationStepImpl_15f25e2b8ea.installStep(InstallationStepImpl_15f25e2b8ea.java)
        at $InstallationStep_15f25e2b8b3.installStep($InstallationStep_15f25e2b8b3.java)
        at ch.tocco.nice2.dbrefactoring.impl.upgrade.DatabaseUpgradeStarter.execute(DatabaseUpgradeStarter.java:52)
        at ch.raffael.hiveapp.impl.EntryPointSupportImpl.run(EntryPointSupportImpl.java:75)
        at ch.raffael.hiveapp.impl.ThreadFactoryImpl$1.run(ThreadFactoryImpl.java:57)
        at java.lang.Thread.run(Thread.java:748)
