.. _Maven_Eirslett:

Maven
=====

.. _maven-eirslett:

Eirslett
--------

.. hint::
    You can disable the maven-frontend-plugin by disabling the eirslett profile, either directly or by passing the
    ``skipEirslett`` argument. Use ``-DskipEirslett`` in the command line to do this.

We use frontend-maven-plugin_ to run various npm tasks such as installing js polyfill packages or packages of the new
client and run gulp to compile Less / Sass and the likes. Since this can take quite a bit of time and does not play well
with multiple processors, all executions of it should be defined in its own profile. This means we can run the plugin
only when we need it.

.. _frontend-maven-plugin: https://github.com/eirslett/frontend-maven-plugin
