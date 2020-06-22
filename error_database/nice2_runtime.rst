Nice2 Runtime Errors
====================

Business Unit Requested not Found (DWR call)
--------------------------------------------

Error
^^^^^

.. code::

    __System.checkHeartbeat.dwr … PermanentPersistException: Expected 1 items but list size is 0!


Cause
^^^^^

This happens when the browser requests a business unit (via ``X-Business-Unit`` HTTP request header) but that
unit doesn't or does no longer exist. You'll see this frequently when you start a different customer on **localhost**
without logging out first.


Solution
^^^^^^^^

Logout and then in again (in all browsers including private browsing windows).


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


Wkhtmltopdf Out-of-Memory
-------------------------

Error:
^^^^^^

    .. code::

        RuntimeException: wkhtmltopdf exited with code 139: Error running wkhtmltopdf

    **Wkhtmltopdf <0.12.4** shows this error:

        .. code::

            java.lang.RuntimeException: Error running wkhtmltopdf


Cause
^^^^^

    The :term:`wkhtmltopdf` process has been killed by signal 9 (SIGKILL) [#f1]_. This most likely happened because
    of an out-of-memory error.


Solution
^^^^^^^^

Option 1: Reduce Memory Usage
`````````````````````````````

* Try to reduce memory usage by …
    * … splitting the report (i.e. select fewer entities at once).
    * … reducing the size of the report resources (e.g. smaller image resolution, no custom font).
    * … simplifying the report (e.g. remove headers/footers, different corporate design).
    * … limiting the number of reports generated concurrently.


Option 2: Increase Available Memory
```````````````````````````````````

Increase **requested memory** to ensure the instance is moved onto a node with enough free memory available and/or adjust
the **memory limit** to ensure it's not hit when generating reports.

See :ref:`nice-memory` for more details [#f2]_.


Full Error:
^^^^^^^^^^^

.. code::

    2017-10-21 01:10:51.683 ERROR nice2.netui.ExceptionHandler [qtp1709207019-7217] user_pk=2996, error_id=7fgDoz, user_name=pgerber, session=117821, clientip=216.239.90.19, request_id=803683
    An exception occurred on the server.
    java.lang.RuntimeException: wkhtmltopdf exited with code 139: Error running wkhtmltopdf: ERROR:
            at ch.tocco.wkhtmltopdf.binary.WkHtmlToPdfBinary.handleError(WkHtmlToPdfBinary.java:92)
            at ch.tocco.wkhtmltopdf.binary.WkHtmlToPdfBinary.run(WkHtmlToPdfBinary.java:74)
            at ch.tocco.nice2.conversion.impl.phantomjs.WkHtmlToPdfConverter.convert(WkHtmlToPdfConverter.java:64)
            at ch.tocco.nice2.conversion.impl.phantomjs.WkHtmlToPdfConverter.convert(WkHtmlToPdfConverter.java:22)
            at $ch.tocco.nice2.conversion.impl.phantomjs.WkHtmlToPdfConverter_15f29845c57.convert(WkHtmlToPdfConverter_15f29845c57.java)
            at $ConverterEngine_15f29845c4a.convert($ConverterEngine_15f29845c4a.java)
            at ch.tocco.nice2.reporting.impl.freemarker.utils.PdfConverterServiceImpl.convertToPdf(PdfConverterServiceImpl.java:54)
            at $ch.tocco.nice2.reporting.impl.freemarker.utils.PdfConverterServiceImpl_15f29845c54.convertToPdf(PdfConverterServiceImpl_15f29845c54.java)
            at $PdfConverterService_15f29845c44.convertToPdf($PdfConverterService_15f29845c44.java)
            at ch.tocco.nice2.reporting.impl.freemarker.handlers.WkHtmlToPdfHandler.handleFileFormat(WkHtmlToPdfHandler.java:36)
            at $ch.tocco.nice2.reporting.impl.freemarker.handlers.WkHtmlToPdfHandler_15f29845c53.handleFileFormat(WkHtmlToPdfHandler_15f29845c53.java)
            at $FileFormatHandler_15f29845c3c.handleFileFormat($FileFormatHandler_15f29845c3c.java)
            at ch.tocco.nice2.reporting.impl.freemarker.FreemarkerReportFactory$FreemarkerReportImpl.processOutput(FreemarkerReportFactory.java:178)
            at ch.tocco.nice2.reporting.impl.freemarker.FreemarkerReportFactory$FreemarkerReportImpl.export(FreemarkerReportFactory.java:136)
            …


.. rubric:: Footnotes

.. [#f1] Processes that exit due to a signal usually exit with a code of 127 + SIGNAL_NUMBER.

.. [#f2] Wkhtmltopdf is a separate process, written in C++ mostly, hence adjusting the Java memory limit won't affect
         it.
