Wkhtmltopdf (WebKit)
====================

`Wkhtmltopdf`_ is an HTML to PDF converter that's based on the WebKit rendering
engine. It is used to create the majority of the PDFs in Nice. In particular,
**wkhtmltopdf is used for all reports that use an output template where the
file format is to "PDF (WebKit)"**.

Implementation
--------------

Within Nice, the class *WkHtmlToPdfConverter* is responsible for calling wkhtmltopdf
(a simple binary). The binary itself is shipped in the `wkhtmltopdf-binary`_ package.


Debugging
---------

Since Nice 2.24 there is a property called ``nice2.conversion.wkhtmltopdf.keepTemporaryFiles``.
If set to ``true``, the intermediate HTML files used to generate a report are kept in the
filesystem permanently.

Enable the property like this::

    oc set env -c nice dc/nice NICE2_APP_nice2.conversion.wkhtmltopdf.keepTemporaryFiles=true

Once enabled, a message is logged for every report generated that looks like this::


    2019-11-29 12:55:47 INFO  - thread: qtp2013329401-745, logger: nice2.conversion.WkHtmlToPdfConverter

    Keeping temporary files after generating a report as requested by the *nice2.conversion.wkhtmltopdf.keepTemporaryFiles* property.

    report title: Education history
    command args: --quiet --title Education\ history --margin-left 0 --margin-right 0 --margin-top 40.0 --margin-bottom 30.0 --disable-smart-shrinking --page-height 297.0 --page-width 210.0 --disable-local-file-access --allow /tmp/1575028546531-0 --allow /tmp/pd4ml/font --header-html /tmp/1575028546531-0/html-to-pdf-header15648480545421830450.htm --footer-html /tmp/1575028546531-0/html-to-pdf-footer9581000101621298200.htm /tmp/1575028546531-0/html-to-pdf-body2944549560328742079.htm /tmp/1575028546531-0/html-to-pdf-out1697885744425284296.pdf

    files:
      body (html):   /tmp/1575028546531-0/html-to-pdf-body2944549560328742079.htm
      header (html): /tmp/1575028546531-0/html-to-pdf-header15648480545421830450.htm
      footer (html): /tmp/1575028546531-0/html-to-pdf-footer9581000101621298200.htm
      output (pdf):  /tmp/1575028546531-0/html-to-pdf-out1697885744425284296.pdf

The html files can now be copied locally for analysis::

    oc cp ${POD_NAME}:/tmp/1575028546531-0/ .

Or, alternatively, you can run wkhtmltopdf in the container to investigate possible conversion issues::

    # extract wkhtmltopdf binary
    unzip -o -d /tmp/ lib/modules/nice2-conversion-module-1.0-SNAPSHOT/lib/ch.tocco.wkhtmltopdf.binary/wkhtmltopdf-binary-*-linux.jar ch/tocco/wkhtmltopdf/binary/wkhtmltopdf

    # run wkhtmltopdf
    /tmp/ch/tocco/wkhtmltopdf/binary/wkhtmltopdf ${ARGS_FROM_INFO_MESSAGE}

**${ARGS_FROM_LOG_MESSAGE}** are the arguments listed as *command args* in the log output
above.  Arguments are escaped for use in the shell. So, just copy and paste.

Once debugging has been completed the property set earlier can be removed again::

    oc set env -c nice dc/nice NICE2_APP_nice2.conversion.wkhtmltopdf.keepTemporaryFiles-



.. _wkhtmltopdf: https://wkhtmltopdf.org/
.. _wkhtmltopdf-binary: https://github.com/tocco/wkhtmltopdf-binary
