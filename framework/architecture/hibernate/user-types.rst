.. _user-types:

Custom user types
=================

Per default Hibernate can map all primitive types (and its wrapper classes) as well as references to other entities.
For all other classes that need to be mapped to the database an :java:extdoc:`UserType<org.hibernate.usertype.UserType>`
must be implemented (for immutable types the base class :java:ref:`ImmutableUserType<ch.tocco.nice2.persist.hibernate.usertype.ImmutableUserType>`
can be used). The user type contains the logic how a specific object should be read from the :java:extdoc:`ResultSet<java.sql.ResultSet>`
and written to the :java:extdoc:`PreparedStatement<java.sql.PreparedStatement>`.

For example the :java:ref:`Login<ch.tocco.nice2.types.Login>` class is mapped with a custom user type (:java:ref:`LoginUserType<ch.tocco.nice2.persist.hibernate.usertype.LoginUserType>`).

*There are two ways to register a new user type:*

It can be registered with a bootstrap contribution (see :ref:`bootstrap`). This way should be used if
there is a distinct object which should be mapped (for example :java:ref:`Login<ch.tocco.nice2.types.Login>`):

.. code-block:: java

    classLoaderService.addContribution(TypeContributor.class, ((typeContributions, serviceRegistry) -> {
        typeContributions.contributeType(new BinaryUserType(binaryAccessProvider, binaryHashingService), Binary.class.getName());
        typeContributions.contributeType(new LoginUserType(), Login.class.getName());
        typeContributions.contributeType(new UuidToStringUserType(), UUID.class.getName());
    }));

The alternative is to bind the user type to a field type (like ``phone``) using a contribution:

.. parsed-literal::

    <contribution configuration-id="nice2.persist.core.UserTypeContributions">
        <contribution type="phone" implementation="service:PhoneUserType"/>
    </contribution>

.. note::

    User types are going to be deprecated in Hibernate 6+ and some refactoring will be required.
    One attempt to replace them with :java:extdoc:`AttributeConverter<javax.persistence.AttributeConverter>`
    was not successful due to classloading issues with hivemind.

Simple user types
-----------------

Most user types are relatively simple and only map between a string or number and an object:

    * :java:ref:`EncodedPasswordUserType<ch.tocco.nice2.security.spi.auth.hibernate.EncodedPasswordUserType>` maps
      instances of :java:ref:`EncodedPassword<ch.tocco.nice2.types.spi.password.EncodedPassword>` from/to a string.
    * :java:ref:`LoginUserType<ch.tocco.nice2.persist.hibernate.usertype.LoginUserType>` maps
      instances of :java:ref:`Login<ch.tocco.nice2.types.Login>` from/to a string.
    * :java:ref:`UuidToStringUserType<ch.tocco.nice2.persist.hibernate.usertype.UuidToStringUserType>` maps
      instances of :java:extdoc:`UUID<java.util.UUID>` from/to a string.
    * :java:ref:`GeolocationTypesContribution<ch.tocco.nice2.optional.geolocation.impl.type.GeolocationTypesContribution>` contains
      user types that support :java:ref:`Latitude<ch.tocco.nice2.types.spi.geolocation.Latitude>` and :java:ref:`Longitude<ch.tocco.nice2.types.spi.geolocation.Longitude>` objects.

``phone`` type
--------------

The :java:ref:`PhoneUserType<ch.tocco.nice2.entityoperation.impl.phone.PhoneUserType>` is applied for all field
of the virtual ``phone`` type.
This user type does not convert between different objects, but formats the phone number using the
:java:ref:`PhoneFormatter<ch.tocco.nice2.toolbox.phone.PhoneFormatter>` whenever a ``phone`` value
is written to the database.

``html`` type
-------------

Like the :java:ref:`PhoneUserType<ch.tocco.nice2.entityoperation.impl.phone.PhoneUserType>`, the
:java:ref:`HtmlUserType<ch.tocco.nice2.persist.hibernate.usertype.HtmlUserType>` does not convert between different objects
but does some string formatting for ``html`` fields.

The formatting behaviour can be contributed using a :java:ref:`HtmlUserTypeExtension<ch.tocco.nice2.persist.hibernate.usertype.HtmlUserTypeExtension>`.
Currently there is only the :java:ref:`PreserveFreemarkerOperatorsHtmlUserTypeExtension<ch.tocco.nice2.templating.impl.freemarker.usertype.PreserveFreemarkerOperatorsHtmlUserTypeExtension>`
which handles escaping in freemarker expressions.

``binary`` type
---------------

The :java:ref:`BinaryUserType<ch.tocco.nice2.persist.hibernate.usertype.BinaryUserType>` handles the :java:ref:`Binary<ch.tocco.nice2.persist.entity.Binary>`
class. The column of a ``binary`` field contains the hash code of the binary and references the ``_nice_binary`` table.

In addition to the mapping of the hash code this user type also calls the configured :java:ref:`BinaryAccessProvider<ch.tocco.nice2.persist.spi.binary.BinaryAccessProvider>`
which stores the binary data if necessary.

User types are also used to map query parameters. If a :java:ref:`Binary<ch.tocco.nice2.persist.entity.Binary>` object is
used as a query parameter, it should obviously not be attempted to write it to the binary data store!
Therefore the binary content is only saved if ``Binary#mayBeStored()`` returns true.
If a hash code is used as a query parameter for a binary field, the string is converted to a :java:ref:`BinaryQueryParameter<ch.tocco.nice2.persist.hibernate.legacy.BinaryQueryParameter>`
by the :java:ref:`StringToBinaryParameterConverter<ch.tocco.nice2.persist.hibernate.legacy.StringToBinaryParameterConverter>`.
``BinaryQueryParameter#mayBeStored()`` returns false so it can safely be used in queries.

See chapter :ref:`large_objects` for more details about large objects.

