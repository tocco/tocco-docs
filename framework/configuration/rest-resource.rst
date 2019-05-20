REST Resource
=============

A REST resource represents a *thing* that can be managed via the REST API. Each module can define its own REST
resources.

.. hint::

   This documentation assumes that you know how REST works and how a REST resource is designed properly. This
   manual only serves as a guide for implementing resources in our application.


.. warning::

   It's very important that every resource is structured in such a way that it fits well into the API as a whole and
   that it follows the REST principles strictly.

   Make sure that your resource is properly designed before you start implementing it, since it's very hard to
   change its design once it's in use (possibly there are also 3rd party applications using the resource).

The steps to implement a REST resource in any module:
    #. Add the required Maven dependency
    #. Extend the Java interface :java:ref:`ch.tocco.nice2.rest.core.spi.RestResource` and define
       your methods and the corresponding JAX-RS annotations.
    #. Implement your interface (by also extending the class :java:ref:`ch.tocco.nice2.rest.core.spi.AbstractRestResource`)
    #. Register your REST resource in the ``hivemodule.xml`` of the module.

The following paragraphs explain in detail how this is done.

.. hint::

   Our REST API is based on the Apache Jersey framework which is an implementation of the JAX-RS specification. This
   documentation only covers the basics of this specification and primarily serves as a guide for implementing
   resources in our application. There is plenty of information publicly available online about Jersey and JAX-RS.

Add Maven Dependency
--------------------

Adding REST resources requires the following dependencies in the ``pom.xml`` of the ``impl`` module.

.. code-block:: XML

    <dependency>
      <groupId>ch.tocco.nice2.rest.core</groupId>
      <artifactId>nice2-rest-core-spi</artifactId>
      <version>${project.version}</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.glassfish.jersey.media</groupId>
      <artifactId>jersey-media-json-jackson</artifactId>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.ws.rs</groupId>
      <artifactId>javax.ws.rs-api</artifactId>
      <scope>provided</scope>
    </dependency>

Additionally the module ``ch.tocco.nice2.rest.core.spi`` has to be imported in the file ``hivemodule.xml``

.. code-block:: XML

   <contribution configuration-id="hiveapp.ClassLoader">
    <import feature="ch.tocco.nice2.rest.core.spi" version="*"/>
   </contribution>

Create Java interface
---------------------

Create the Java interface for your resource by extending :java:ref:`ch.tocco.nice2.rest.core.spi.RestResource`.

The following interface defines a REST resource which will be available on ``${BASE_PATH}/events/{city}``.
It defines a method called ``getEvents()`` and a second method ``addEvent()``. The first method is mapped to
`GET` requests, the second one to `POST` requests. Both methods return JSON results.

.. hint::

   ${BASE_PATH} is ``/nice/rest``. So the full path of the following resource is ``/nice2/rest/events/{city}``.

.. code-block:: Java

   @Path("/events/{city}")
   public interface EventsResource extends RestResource {

       @GET
       @Produces(MediaType.APPLICATION_JSON)
       @Operation(
           summary = "Load events",
           description = "Load events which take place in a certain city",
           tags = "events"
       )
       CollectionBean getEvents(
           @PathParam("city") @Parameter(description = "name of the city") String city,
           @QueryParam("sort") @Parameter(description = "comma separated string of fields to sort by") String sort
       );

       @POST
       @Consumes(MediaType.APPLICATION_JSON)
       @Produces(MediaType.APPLICATION_JSON)
       @Operation(
           summary = "Create event",
           description = "Create a new event",
           tags = "events"
       )
       Response addEvent(EventBean event)
   }

There is an extensive set of **JAX-RS** annotations which can be used to define the behavior of a resource:

.. list-table::
   :header-rows: 1

   * - Annotation
     - Description
   * - Path
     - Identifies the URI path. Can be specified on a class or a method.
   * - PathParam
     - Represents the parameter of the URI path.
   * - GET
     - Specifies the method that responds to GET requests.
   * - POST
     - Specifies the method that responds to POST requests.
   * - PUT
     - Specifies the method that responds to PUT requests.
   * - ch.tocco.nice2.rest.core.spi.PATCH
     - Specifies the method that responds to PATCH requests (note that this annotation is not part of the
       ``javax.ws.rs`` package).
   * - HEAD
     - Specifies the method that responds to HEAD requests.
   * - DELETE
     - Specifies the method that responds to DELETE requests.
   * - OPTIONS
     - Specifies the method that responds to OPTIONS requests.
   * - FormParam
     - Represents the parameter of the form.
   * - QueryParam
     - Represents the parameter of the query string of an URL.
   * - HeaderParam
     - Represents the parameter of the header.
   * - CookieParam
     - Represents the parameter of the cookie.
   * - Produces
     - Defines the media type for the response such as XML, PLAIN, JSON etc.
   * - Consumes
     - Defines the media type that the method of a resource class can consume.

Swagger documentation
^^^^^^^^^^^^^^^^^^^^^

There is a Swagger documentation available on ``/nice2/swagger``. Use the annotations ``@Operation`` and ``@Parameter``
to describe the resource in this documentation.

See the `Swagger API documentation`_ for more information about that.

.. _Swagger API documentation: https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations

Versioning
^^^^^^^^^^

If you have to introduce a breaking change in our REST API, use the annotations ``ch.tocco.nice2.rest.core.spi.Before``
and ``ch.tocco.nice2.rest.core.spi.Since`` to change the behavior in a specific Nice version and leave the old
behavior in place for older versions. This ensures that all clients which use the API in combination with a specific
version number don't break.

.. warning::

   Keep in mind that we should maintain backward compatibility in our REST API whenever possible. Never forget
   that there are several third parties using our API.

Implement resource
------------------

Add the implementation for your resource by implementing your created interface and extending
:java:ref:`ch.tocco.nice2.rest.core.spi.AbstractRestResource`.

.. code-block:: Java

   public class EventsResourceImpl extends AbstractRestResource implements EventsResource {
       @Override
       public CollectionBean getEvents(String city, String sort) {
           // load events here and return response
       }

       @Override
       public Response addEvent(EventBean event) {
           // create event here and return response
       }
   }

How to test your resource
^^^^^^^^^^^^^^^^^^^^^^^^^

Test your resource by extending :java:ref:`ch.tocco.nice2.rest.testlib.AbstractInjectingJerseyTestCase`. Writing
tests for your resource by extending this base class allows you to implement **end-to-end** tests which test the
whole process including routing (via JAX-RS annotations on your interface) and error handling (via the exception
mappers you contribute in the test).

.. hint::

   Compared to simple unit tests, this is the preferred way to test your resource. However, lower level unit tests
   are important as well.

Set up your test like any conventional :java:ref:`ch.tocco.nice2.persist.testlib.inject.AbstractInjectingTestCase`
and additionally implement the abstract method :java:ref:`getRestResources():List<?>` and optionally
:java:ref:`getExceptionMappers():List<ExceptionMapper>` to test error handling.

First add the required test dependency in your ``pom.xml``:

.. code-block:: XML

   <dependency>
     <groupId>ch.tocco.nice2.rest.testlib</groupId>
     <artifactId>nice2-rest-testlib</artifactId>
     <version>${project.version}</version>
     <scope>test</scope>
   </dependency>

Then add your test class(es):

.. code-block:: Java

   import javax.ws.rs.client.Entity;
   import javax.ws.rs.core.MediaType;
   import javax.ws.rs.core.Response;
   import javax.ws.rs.ext.ExceptionMapper;

   import import ch.tocco.nice2.rest.testlib.AbstractInjectingJerseyTestCase;

   public class AddEventTest extends AbstractInjectingJerseyTestCase {
       @Resource
       private EventsResourceImpl eventsResource;
       @Resource
       private List<ExceptionMapper> exceptionMappers;

       @Override
       protected void setupTestModules() {
           install(FixtureModules.embeddedDbModules(false));
           install(FixtureModules.createSchema());
           install(RestCoreModules.main());
           bind(EventsResource.class, EventsResourceImpl.class);
           bindDataModel(MyTestDataModel.class);
       }

       @Override
       protected List<?> getRestResources() {
           return ImmutableList.of(
               eventsResource
           );
       }

       @Override
       protected List<ExceptionMapper> getExceptionMappers() {
           return exceptionMappers;
       }

       @Test
       public void testAddEvent() throws Exception {
           Entity entity = Entity.entity(new EventBean(), MediaType.APPLICATION_JSON_TYPE);
           Response response = target("/events/zurich").request().post(entity);
           assertEquals(response.getStatus(), 201);

           String location = response.getHeaderString("Location");
           assertNotNull(location);

           assertEventExists(URI.create(location));
       }
   }

.. warning::

   When targeting an url with query parameters, the query params should not be added to the path but attached with
   `.queryParams` or the response will most likely be `404 - Not Found`.

   **NO** 

   :java:ref:`Response response = target("/location/suggestions?city=Züri").get();`

   **YES** 

   :java:ref:`Response response = target("/location/suggestions").queryParam("city", "Züri").request().get();`

   

Register resource
-----------------

The resource needs to be registered as hivemind service in the file ``hivemodule.xml``.

.. code-block:: XML

   <service-point id="EventsResource" interface="ch.tocco.nice2.[...].EventsResource">
     <invoke-factory>
       <construct class="ch.tocco.nice2.[...].EventsResourceImpl"/>
     </invoke-factory>
   </service-point>

Now the service needs to be contributed as REST Resource.

.. code-block:: XML

   <contribution configuration-id="nice2.rest.core.Resources">
    <resource resource-id="EventsResource"/>
   </contribution>

Use it
------

Now start the application and send an HTTP request to `${HOST}/nice2/rest/events/zurich`. If you send a GET request
(i.e. by simply entering the URL in your browser), ``getEvents()`` should be called and you should receive a JSON
representation of events which take place in Zürich.

Enable cross-origin access (optional)
-------------------------------------

By default, the REST resources cannot be accessed from another domain outside the domain from which the REST API is
served (forbidden by the `same-origin security policy`_).

Follow the steps described in :doc:`../rest/cors/index` if access from other domains should be enabled.

.. _same-origin security policy: https://en.wikipedia.org/wiki/Same-origin_policy
