Swagger Integration
===================

`Swagger`_ is used for the REST-API documentation. It can be requested on the following URL: ``https://<INSTALLATION>/nice2/swagger``.

Swagger works with the `OpenAPI Specification`_. In order to work correctly all REST resources must be described by
`Swagger Annotations`_ out of which the OpenAPI specification is generated.

Swagger is integrated in the ``nice2-rest-doc`` module.

Swagger is integrated as follows
--------------------------------

1. Swagger-UI as Static Resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`Swagger UI`_ is a tool to generate a single page which visualizes the REST resources. To generate the page download
the latest version of `Swagger UI`_, open a terminal and go to the downloaded folder. Run the following commands:

 - ``npm install``
 - ``npm build``

The generated page now should be inside the ``dist`` folder.

.. warning::
    The commands could be different in newer versions. Check the `scripts page`_ for more information.

The generated page is copied to ``PATH/TO/NICE2/rest/doc/module/resources/webapp/swagger``. In the hivemodule.xml this
path is defined as static resource.

.. code-block:: XML

   <contribution configuration-id="hiveapp.http.Resources">
     <mount ctx-id="root" path="/nice2/swagger" resource="[#self]/webapp/swagger"/>
   </contribution>

Edit the ``index.html`` and replace the value of the `url` property with ``"/nice2/openapi"``. This url tells Swagger
where to fetch the `OpenAPI Specification`_.

Additionally set all links to the generated JS and CSS files correctly. For example replace ``href="./swagger-ui.css"``
with ``href="swagger/swagger-ui.css"``

2. NiceOpenApiServlet
^^^^^^^^^^^^^^^^^^^^^

The :abbr:`NiceOpenApiServlet (ch.tocco.nice2.rest.doc.impl.NiceOpenApiServlet)` extends the
:abbr:`OpenApiServlet (io.swagger.v3.jaxrs2.integration.OpenApiServlet)` which produces the `OpenAPI Specification`_
for the Swagger-UI page.

In the :abbr:`NiceOpenApiServlet (ch.tocco.nice2.rest.doc.impl.NiceOpenApiServlet)` the following is configured:
 - All REST resources
 - Some information like the company and contact data
 - The authentication mechanism (so that authenticated requests can be executed)

The serlvet can be requested under ``<INSTALLATION>/nice2/openapi``.

3. Describe REST Resources with Swagger Annotations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To describe the REST resources `Swagger Annotations`_ are used. The
:abbr:`NiceOpenApiServlet (ch.tocco.nice2.rest.doc.impl.NiceOpenApiServlet)` converts these to the `OpenAPI Specification`_.

Check the `Swagger Annotation docs`_ to see how resources can be described.

Example:

.. code-block:: Java

   @Path("/entities/{name}")
   public interface EntitiesResource extends RestResource {

       @GET
       @Produces(MediaType.APPLICATION_JSON)
       @JacksonFeatures(serializationEnable = {SerializationFeature.INDENT_OUTPUT})
       @Operation(
           summary = "Load entities",
           description = "Load entities of a given model with paging and optional sorting",
           tags = "entities"
       )
       CollectionBean getEntities(
           @PathParam("name") @Parameter(description = "name of the entity model to load") String modelName,
           @QueryParam("_offset") @Parameter(description = "offset from first result for paging") String offset,
           @QueryParam("_limit") @Parameter(description = "max amount of entities to load") String limit,
           @QueryParam("_sort") @Parameter(description = "comma separated string of fields to sort by") String sort
       );
   }


.. _Swagger: https://swagger.io/
.. _OpenAPI Specification: https://en.wikipedia.org/wiki/OpenAPI_Specification
.. _Swagger UI: https://swagger.io/tools/swagger-ui/
.. _Swagger Annotations: https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations
.. _scripts page: https://github.com/swagger-api/swagger-ui/blob/master/docs/development/scripts.md
.. _Swagger Annotation docs: https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations
.. _Swagger Annotations: https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations
