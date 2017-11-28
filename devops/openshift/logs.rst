Logs
====

.. _Kibana User Guide: https://www.elastic.co/guide/en/kibana/current/index.html
.. _Discover: https://www.elastic.co/guide/en/kibana/current/discover.html
.. _Lucene Query Syntax: https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-query-string-query.html#query-string-syntax

Getting Started
---------------

1. Log in on https://logging.appuio.ch.

.. figure:: logs/main_page.png
   :scale: 80%

2. Open *saved searches* menu

3. Select a *saved search*

   Select ``Nice`` to get the Nice event logs or ``Nginx`` to get the Nginx access logs.

4. Show all projects

5. Select Project

   There is one project per installation.

6. Select a time frame

.. warning::

   When selecting a *saved search* (step 2) you're automatically switched to the project that's associated with the
   search. Ensure it is the right project (step 4).

.. hint::

   For more information have a look at `Kibana User Guide`_, in particular the `Discover`_ section and
   `Lucene Query Syntax`_.


Query Syntax
------------

.. figure:: logs/query.png
   :scale: 80%

1. Search for **ModelException** in field *stack_trace*.

2. The word **ERROR** must be contained in the result. Without the **+**, the results matching closest the query are
   returned even if they don't contain all search terms.

3. Filter out all results that contain the word **runtime**.

4. Search for the phrase **some text** rather than the words **some** and **text**. You may also have to use quotes if
   the search term contains special characters. For instance, if it contains a hyphen, like **start-up**, it is treated
   as two words. Using ``"start-up"`` avoids this. In case you want to search a particular field, use
   ``message:"my search term"``.

More about `Lucene Query Syntax`_

Add Columns
-----------

.. figure:: logs/add_column.png
   :scale: 80%

You can use the panel on the left or the detail view to show more columns.


Filter by Fields
----------------

.. figure:: logs/filter_by_field.png
   :scale: 80%

By using the +/- magnifying glasses in the detail view, you can filter based on a field's value.


Nice Logs
---------

Filter by Log Level
^^^^^^^^^^^^^^^^^^^

By default only level ERROR and WARN are shown. Toggle as shown below to change that.

.. figure:: logs/nice_log_level.png
   :scale: 80%

   Level DEBUG and INFO are hidden, solid red, and WARN and ERROR are shown, diagonally stripped.


Searching for Similar Exceptions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: logs/stack_hash.png
   :scale: 80%

1. Open entry detail

2. Filter by ``stack_hash`` [#f1]_


.. rubric:: Footnotes

.. [#f1] `Details About Stack Hashes <https://github.com/logstash/logstash-logback-encoder/blob/master/stack-hash.md>`__.
