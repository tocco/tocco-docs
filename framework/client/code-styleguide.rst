.. _Coding-Styleguide:

Coding Styleguide
=================

General
-------

ESlint
^^^^^^
Most code styles are enforced by eslint. The Travis CI build
will fail if not every ESlint rule is fulfilled. Active ESlint rules can
be seen here:
https://github.com/tocco/tocco-client/blob/master/.eslintrc


To show current linting errors and warnings:

.. code-block:: console

  yarn lint


To try auto-fix them:

.. code-block:: console

  yarn lint:fix


.. note::

  Eslint command will run as a git pre commit hook. It isn't possible to commit anything as long as there are linting errors.
  Lint will also be executed automatically on our CI.


**Setup Linting with IntelliJ**

* Install ESLint Plugin
* Settings (Preferences on OS X) | Languages & Frameworks | JavaScript |  Code Quality Tools --enable
* Settings (Preferences on OS X) | Editor | Inspections | Code Style Issues | Unterminated statement -- disable


Folders and structure
---------------------

Naming conventions for folders
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

========== ====== ==============
What       Case   Example
========== ====== ==============
Packages   Kebab  entity-browser
Components Pascal SearchField
Module     Camel  searchForm
Any other  Kebab  test-data
========== ====== ==============

Tocco variable naming
^^^^^^^^^^^^^^^^^^^^^
To be consistent trough all apps following variable names should be chosen when dealing with the nice backend:

+-------------+------------------+---------------------------------------------------------------------------------------+
| Name        | Alternative name | Description                                                                           |
+=============+==================+=======================================================================================+
| entityName  | name             | Technical entity name e.g. User or Education_schedule                                 |
+-------------+------------------+---------------------------------------------------------------------------------------+
| entityLabel | label            | Localized entity label  e.g. Person or Lehrplan                                       |
+-------------+------------------+---------------------------------------------------------------------------------------+
| entityModel | model            | Object containing the whole model including the name, label and fields                |
+-------------+------------------+---------------------------------------------------------------------------------------+
| entityKey   | key              | Primary key of entity. Avoid "id" or "pk" as substitute                               |
+-------------+------------------+---------------------------------------------------------------------------------------+
| entityId    | id               | Object containing entityName and entityKey eg. {entityName: 'User, entityKey: "33"}   |
+-------------+------------------+---------------------------------------------------------------------------------------+
| formName    |                  | Form name including the scope e.g. User_list                                          |
+-------------+------------------+---------------------------------------------------------------------------------------+
| formBase    |                  | Only the form base without the scope e.g. UserSearch                                  |
+-------------+------------------+---------------------------------------------------------------------------------------+
| form        |                  | Form object containing name and fields                                                |
+-------------+------------------+---------------------------------------------------------------------------------------+

Javascript
----------
Actions
^^^^^^^

-  Wrap arguments in payload attribute
-  Use arrow functions
-  Returning object literals (no return statement used)

.. code-block:: js

       // Good
       export const setPending = (pending = false) => ({
         type: SET_PENDING,
         payload: {
           pending
         }
       })

       // Not so good
       export function setPending(pending = false) {
         return {
           type: SET_PENDING,
           pending: pending
          }
       }

Reducers
^^^^^^^^^

-  Use arrow functions
-  Use destructuring assignment

.. code-block:: js

     // Good
     const updateOldPassword = (state, {payload}) => ({
       ...state,
       oldPassword: payload.oldPassword
     })

     // Not so good
     function updateOldPassword(state, args) {
       return Object.assign({}, state, {
         oldPassword: args.payload.oldPassword
       })
     }

Tests
^^^^^

-  Group tests hierarchically according to directory structure starting
   with the package-name
-  *test* description should always start with ``should``

.. code-block:: js

     // Good
     describe('package-name', () => {
       describe('components', () => {
         describe('Image component', () => {
           test('should render an image', () => {
             //...

     // Bad
     describe('Image component', () => {
        test('renders an image', () => {
           //...

-  Use Chai to.be.true instead of equal(true)

.. code-block:: js

     // Good
     expect(withTitle.find(LoginFormContainer).prop('showTitle')).to.be.true

     // Not so good
     expect(withTitle.find(LoginFormContainer).prop('showTitle')).to.equal(true)

-  If enzyme is used to load a component, name the variable ``wrapper``
   whenever possible

.. code-block:: js

     // Good
     const wrapper = shallow(<Foo onButtonClick={onButtonClick} />)