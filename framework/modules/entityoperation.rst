Entityoperation
===============

Merge
-----

Per default all fields and relations from the detail form plus the relations tabs are considered for merging.
If a user does not have write access for a relation (or for some related entities) a to-many relation is automatically moved and is not displayed.

There are some simple ``RelationMergeHandler`` implementations such as ``ToManyRelationMergeHandler``,
``ToManyRelationMergeHandler`` or ``PrivilegedRelationMergeHandler`` which process the merging.
The handler with the highest ``priority()`` where ``supports(RelationModel)`` returns ``true`` is executed.
However sometimes you must implement a custom relation handler to get a useful result
(e.g. ``EntityDocsRelationMergeHandler`` or ``AddressAddressRelationMergeHandler``).

Some relations should not be visible in the merge action. In the following example the ``relMark`` relation on the ``User`` entity and the
``relDebitor_information`` relation on the ``Address`` entity are not displayed.
But the ``relMark`` is automatically moved because the optional argument ``blindCopy`` is true while the ``relDebitor_information`` is not touched.

.. code-block:: xml
        :caption: Excluded relation

        <contribution configuration-id="nice2.entityoperation.ExcludeRelations">
            <excludeRelation modelName="User" relationName="relMark" blindCopy="true"/>
            <excludeRelation modelName="Address" relationName="relDebitor_information"/>
        </contribution>

Some fields should not be visible in the merge action. In the following example the ``longitude`` field on the ``Address`` entity
is not displayed.

.. code-block:: xml
        :caption: Excluded field

          <contribution configuration-id="nice2.entityoperation.ExcludeFields">
            <excludeField modelName="Address" fieldName="longitude"/>
          </contribution>

If a user does not have write permission for a relation, moving the related entities to the target entity is not possible.
However there is the option to set a relation as privileged. In such a case the relation is moved without checking the permission of the user.

.. code-block:: xml
        :caption: Privileged relation

        <contribution configuration-id="nice2.entityoperation.MergeRelationsPrivileged">
            <merge-relation-privileged modelName="User" relationName="relCorrespondence"/>
        </contribution>

For some relations either all or no related entities should be moved.
In the following example the relation ``relRegistration`` on the entity ``User`` is defined as an "all-or-nothing-relation".
Additionally the entity ``Resource`` is always an "all-or-nothing" relation.
So during merging ``User``, ``Address`` etc. entities a single resource can never be selected/deselected.
Either all resources of a target entity are move or nothing is moved.

.. code-block:: xml
        :caption: All or nothing relation

        <contribution configuration-id="nice2.entityoperation.AllOrNothingRelation">
                <all-or-nothing-relation entityModel="User" relation="relRegistration"/>
                <all-or-nothing-relation entityModel="Resource"/>
        </contribution>

Per default nothing happens with the source entity. However there is an option to delete the source entity.
In the following example all ``Address`` source entities are deleted after merging.

.. code-block:: xml
        :caption: Delete source entity

        <contribution configuration-id="nice2.entityoperation.DeleteStrategies">
            <delete-strategy entityModel="Address"/>
        </contribution>

Another option is to change a related (lookup) entity.
In the following example the ``relAddress_status`` relation is set to ``archive`` for the source entities.

.. code-block:: xml
        :caption: Archive source entity (set a related entity)

        <contribution configuration-id="nice2.entityoperation.SourceRelationStrategies">
            <source-relation-strategy entityModel="Address" relation="relAddress_status" keyField="unique_id" value="archive"/>
        </contribution>

Last there is the option to change a field value.

.. code-block:: xml
        :caption:  Archive source entity (change field value)

        <contribution configuration-id="nice2.entityoperation.SourceFieldStrategies">
            <source-field-strategy entityModel="Address" field="callname" value="archive"/>
        </contribution>