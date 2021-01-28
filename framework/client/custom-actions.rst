Custom Actions
==============
Custom actions are independent applications that can be executed in a modal window, as fullscreen actions or standalone (e.g. old client).
The API should be consistent and is explained here.

Input
-----

.. list-table::
   :header-rows: 1
   :widths: 10 20 20

   * - Name
     - Description
     - Example
   * - selection
     - Selected entities. Can either be an array of keys (type = id) or a query.  
     - | keys

       .. code-block:: json 

          {                                                          
            "count": 2,                                               
            "entityName": "User",                                     
            "ids": [                                                  
              "11081",                                                 
              "11092"                                                  
            ],                                                        
            "length": 2,                                              
            "type": "ID"                                              
          }        

       | query

       .. code-block:: json                                         
                                                                                                                  
        {                                                          
          "count": 77,                                              
          "entityName": "User",                                     
          "query": {                                                
            "filter": ["user_active"],                               
            "where": "(fulltext(\"Tocco\") or fulltext(\"Tocco*\"))",
            "hasUserChanges": true                                   
            },                                                       
          "type": "QUERY"                                           
        }                                                                                                            
   * - context
     -  | Description: Can be passed by the app calling the action to provide some context information.
        | As of now the viewName and formName are passed. But this object can be extended as needed.
     -  | entity-list
     
        .. code-block:: json

          {
          "viewName": "list",   
          "formName": "UserSearch"
          }

        | entity-detail

        .. code-block:: json

          {
          "viewName": "detail",
          "formName": "UserSearch"
          }
   * - navigationStrategy
     - An object containing utils to add navigation functionality. See `Github File`_ for more details. 
     -

.. _Github File: https://github.com/tocco/tocco-client/blob/master/packages/tocco-util/src/navigationStrategy/navigationStrategy.js

Callback Events
---------------
This functions are also passed to the action via input and should give feedback about the exit status.

.. list-table::
   :header-rows: 1
   :widths: 10 20 

   * - Name
     - Params object  
   * - onSuccess
     - | *message {string}* optional string that is shown in an successful info box.  
       | *remoteEvents {array}* List of remote events :ref:`Remote Events`     
   * - onError 
     - *message {string}* optional string that is shown in an error info box.     
   * - onCancel
     - 


.. _Remote Events:

Remote Events
^^^^^^^^^^^^^^
Indicates what changed within the execution of an action. This helps the executing app to e.g. refresh or redirect.
At the moment a remote event consists of one of three types and the affected entities.

Types: 'entity-create-event', 'entity-delete-event', 'entity-update-event'

 .. code-block:: json           

  {
   "type": "entity-update-event",
   "payload": {
    "entities": [
       {"entityName": "User", "key": "1"}
      ]
    }
  }


