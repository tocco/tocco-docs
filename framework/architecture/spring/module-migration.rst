Module migration
================

This guide explains how to migrate an existing Tocco module from HiveApp to Spring Boot.

Create new module
-----------------

Apart from a few base modules, all modules should be created in either the ``core``, ``optional`` or ``customer``
directories. Simply create a new directory named after the module (e.g. ``persist-core``) and create an empty
``build.gradle`` file inside it.

The new module needs to be registered in the ``settings.gradle`` file in the root directory of the project:

  .. code-block:: text

    include 'core:<module-name>'

At this point it is recommended to refresh the Gradle project in IDEA in order to have IDE support for the next steps.

We use the `Java Platform Module System <https://www.oracle.com/corporate/features/understanding-java-9-modules.html>`_
for modularization. First we need to create the module descriptor ``module-info.java`` in the main source directory
``src/main/java``:

  .. code-block:: java

    open module nice.core.reporting {

    }

The module name should be in the following format: ``nice.core/optional/customer.<module-name>``. Note that the module
must be ``open``. This allows other modules to access the module classes by reflection, which is necessary for
the discovery of the beans by spring. It would also be possible to only open certain packages to certain modules
using the ``opens <package> to <module>`` syntax.

If a customer module is migrated a main class must be created additionally:

  .. code-block:: java

    @SpringBootApplication(scanBasePackages = "ch.tocco.nice2")
    public class Nice2Application {

    	public static void main(String[] args) {
    		SpringApplication.run(Nice2Application.class, args);
    	}
    }

In addition the module  must be registered in the ``build.gradle`` (see `https://docs.gradle.org/current/userguide/application_plugin.html#sec:application_modular <https://docs.gradle.org/current/userguide/application_plugin.html#sec:application_modular>`_):

  .. code-block:: groovy

    bootRun {
        mainModule.set("nice.customer.test")
    }

Adding sources
--------------

In the next step the sources files may be added to the new module. All ``.java`` files should be placed in the
``src/main/java`` directory, all ``.groovy`` files must be placed in the ``src/main/groovy`` (which also supports
java files).

Most existing modules already have reasonable package names, but if not, now would be a good time to adjust them.

A module no longer contains the ``api``, ``impl`` and ``module`` sub-modules.
The ``api`` sources (which should be accessible by other projects) should be placed in specific sub-package
(ideally also called ``api``). These packages then have to be declared in the module-descriptor:

  .. code-block:: java

    exports <package-name>;

Only exported packages may be read by other modules.
The ``impl`` sources may be moved to any other package that is not exported.

The test sources belong in the ``src/test/java`` (or ``src/test/groovy``) directories. Test sources typically
do not have a ``module-info.java`` class, which means that internal classes of other modules are also accessible
in tests. However it is possible to add a module descriptor if the modularization is required for integration tests.

Adding dependencies
-------------------

After moving all the source files, some dependencies will most likely be missing.
Dependencies may be declared in the ``build.gradle`` file of the module:

  .. code-block:: groovy

    dependencies {
        api project(":core:model:entity")

        implementation project(":core:reporting")

        testImplementation "org.testng:testng"
    }

See also the gradle `documentation <https://docs.gradle.org/current/userguide/java_library_plugin.html>`_ for more details.

    * The ``api`` dependencies are transitive and are automatically available for all modules that depend on this module.
      This should be used if the dependency is required by a class in the API package (that is exported from the module).

    * ``implementation`` should be used for all other dependencies that are only used internally.

    * Each (non-test) dependency also requires an entry in the ``module-info.java`` file:
      ``requires transitive nice.core.model.entity;`` (transitive only for api dependencies). See the `gradle documentation <https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_modular>`_
      for details.

When adding project dependencies keep in mind that it's not necessary to add every single dependency because
of the transitive api dependencies.

External dependencies should be referenced without an explicit version number. Library versions are managed in the ``dependencyManagement`` block
in the root ``build.gradle``.

Dependencies which should be available in all modules (like guava for example) should be declared in the
``dependencies`` block of the root ``build.gradle``. The corresponding ``module-info.java`` entry
should be made in the ``boot`` module (transitive) which is available in all modules.

Some external dependencies might be problematic, if they have not been modularized properly:

    * If the library is not a module and doesn't have an automatic module name
    * Split packages: a certain package may only be used by one library. This often happens with ``javax.*`` packages.

The `extraJavaModuleInfo Gradle plugin <https://github.com/jjohannes/extra-java-module-info>`_ may be used to fix these issues (see root ``build.gradle``).

Adding resources
----------------

Normal classpath resources can be placed in the ``src/main/resources`` directory as usual. Keep in mind that the
modularization is also applied to the resources and make sure that the correct packages are used.

The resources that used to be in the ``module`` sub-module are handled differently. They should be placed
in the ``resources`` directory of the module (using the same internal structure as before).
During the build these resources will be moved to the ``src/main/resources/META-INF`` directory. This is necessary
because the META-INF directory is excluded from modularization. Otherwise the compiler would complain about
using the same 'package' (e.g. ``model.entities``) in multiple modules.

The paths that are moved automatically are defined by the ``ext.resourceIncludePattern`` property of the root ``build.gradle``.
Additional paths can be added for a specific module by adding the following to its ``build.gradle``:

  .. code-block:: groovy

    resourceIncludePattern << '...'

Migrating the hivemodule.xml
----------------------------

The first step would be running the ``HiveappModuleMigrator`` class which takes three arguments:

    * path to hivemodule.xml file that should be migrated
    * path to the new module that is being migrated
    * base package name of the new module

This script creates spring configuration classes for contributions that can be easily migrated.
It also creates a file called ``hivemodule.replaced.xml`` which only contains the contributions and services
which still need to be migrated manually.

The remaining elements should be migrated in the following order:

Configuration-Point
^^^^^^^^^^^^^^^^^^^

There are a few different options how to migrate configuration points:

  .. code-block:: xml

    <contribution configuration-id="nice2.persist.core.HibernateBootstrapContributions">
      <contribution implementation="service:GeolocationTypesContribution"/>
    </contribution>

The above example only contributes a service.
The only thing to do here is to annotate the setter method with  ``@Autowired`` where the configuration
should be injected. Instead of the using a setter it's also possible to use the constructor for injection.

The approach above only works if the different contributions implement the same interface.
If the contributions do not implement a common interface, an annotation can be used instead
(have a look at `this commit <https://gitlab.com/toccoag/spring-boot-test/-/commit/9df5ba92ca6ca66c3339bcd69ad73f2e6ade725c>`_
to see how to use annotations for this).


  .. code-block:: xml

    <contribution configuration-id="nice2.reporting.Reports">
      <report id="report_name"
              outputTemplate="template_name"
              synchronize="true"
              label="report.label">
      </report>
    </contribution>

For the above case a contribution class that contains these properties needs to be created (often
such a class already exists and can be reused). A list of this class can then be autowired into the
target (as described above). Note that the class must be in an exported package, as it needs to be
accessible to modules that want to contribute.
Consider extending the ``HiveappModuleMigrator`` for such cases.

  .. code-block:: xml

    <contribution configuration-id="Functions">
      <function name="DATETIMEADD" function="service:DatetimeAddFunction"/>
    </contribution>

This example is a mix of the first two examples, it contains both a service and some additional information.
There are two different ways to migrate these cases:

    * Using a contribution class like in the second example
    * Using a custom annotation. The service can be autowired as described in the first example, the metadata
      can then be read from the annotation in the setter or constructor. Note that if a qualifier annotation is used
      to inject the beans, it cannot be used to add metadata. An additional annotation needs to be used.

Services
^^^^^^^^

It is usually sufficient to annotate the service implementation with the ``@Component`` annotation.

  .. code-block:: xml

    <set-configuration configuration-id="ServicePointCategoryExtractors" property="categoryExtractors"/>

If a configuration-point is injected into the service the setter has to be annotated with ``@Autowired``
or the property has to be moved into the constructor.

  .. code-block:: xml

    <set property="enabled" value="${nice2.metrics.enabled}"/>

Setter for properties can be removed and replaced with the ``@Value("${..}")`` annotation directly on the field.


Contributions
^^^^^^^^^^^^^

All configuration points for these contributions should have already been migrated, otherwise the migration order is wrong.

How contributions are migrated depends on how the corresponding configuration point was migrated.

    * If only a service is contributed it is sufficient to add the ``@Component`` annotation to the contribution class
      (or the qualifier annotation in case it is used)
    * If there is an additional metadata annotation it needs to be placed on the class as well
    * If a custom contribution class is used, an instance of this class needs to be returned from a
      method that is annotated with ``@Bean`` and is in class that is annotated with ``@Configuration``
