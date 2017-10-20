Nice2 Runtime Errors
====================

Business Unit Requested not Found (DWR call)
--------------------------------------------

Error
^^^^^

.. code::

    \__System.checkHeartbeat.dwr … PermanentPersistException: Expected 1 items but list size is 0!


Solution
^^^^^^^^

Frankly, **Logout and then in again.**

In detail, this happens when the browser request a business unit (via ``X-Business-Unit`` HTTP request header) but that
unit doesn't or does no longer exist. You'll see this frequently when you start a different customer on **localhost**
without logging out first.


Full Error
^^^^^^^^^^

.. code::

    2017-10-20 14:31:17.433 WARN  org.eclipse.jetty.servlet.ServletHandler [qtp1259602878-119]
    /nice2/dwr/call/plaincall/__System.checkHeartbeat.dwr
    ch.tocco.nice2.persist.PermanentPersistException: Expected 1 items but list size is 0!
        …
        at ch.tocco.nice2.businessunit.impl.BusinessUnitManagerImpl.getBusinessUnitById(BusinessUnitManagerImpl.java:309) ~[na:na]
        at $ch.tocco.nice2.businessunit.impl.BusinessUnitManagerImpl_15f39c09bac.getBusinessUnitById(BusinessUnitManagerImpl_15f39c09bac.java) ~[na:na]
        at $BusinessUnitManager_15f39c097a7.getBusinessUnitById($BusinessUnitManager_15f39c097a7.java) ~[na:na]
        at ch.tocco.nice2.businessunit.impl.SetBusinessUnitFilter.getBusinessUnit(SetBusinessUnitFilter.java:58) ~[na:na]
        at ch.tocco.nice2.businessunit.impl.SetBusinessUnitFilter.doFilter(SetBusinessUnitFilter.java:41) ~[na:na]
        at $ch.tocco.nice2.businessunit.impl.SetBusinessUnitFilter_15f39c09bef.doFilter(SetBusinessUnitFilter_15f39c09bef.java) ~[na:na]
        …
