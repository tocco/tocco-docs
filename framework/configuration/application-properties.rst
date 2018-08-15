.. _application-properties:

Application Properties
======================

Application properties are used to configure an installation. Application properties are defined in the file
``application.properties``. These files are located under ``path/to/nice2/customer/CUSTOMER_NAME/etc``.

Application properties can be overwritten in a file ``application.local.properties`` also located in the same directory.
This can be useful to test something locally or if the properties are sensitive (e.g. usernames and passwords).
An ``application.local.properties`` file is ignored by git (.gitignore) and must be created manually.

Application properties are passed to hivemind services which can use them in their logic.

Create a New Application Property
---------------------------------

Application properties can be passed to hivemend services. Lets look at the ``SmsParameterProvider``. The service
without any application properties would be defined as follows in the ``hivemodule.xml`` file.

.. code-block:: XML

   <service-point id="SmsParameterProvider" interface="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProvider">
     <invoke-factory>
       <construct class="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProviderImpl"/>
     </invoke-factory>
  </service-point>

In order to work properly the service needs five properties defined by application properties (provider, username,
password, warningLimit, simulateSms). To achieve this the service has to be defined as follows.

.. code-block:: XML

   <service-point id="SmsParameterProvider" interface="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProvider">
     <invoke-factory>
       <construct class="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProviderImpl">
         <set property="provider" value="${sms.provider}"/>
         <set property="username" value="${sms.provider.username}"/>
         <set property="password" value="${sms.provider.password}"/>
         <set property="warningLimit" value="${sms.warning.limit}"/>
         <set property="simulateSms" value="${sms.simulateSms}"/>
       </construct>
     </invoke-factory>
  </service-point>

When hivemind injects the service it looks for setter methods for these properties. The Service Implementation must
provide these. For each property a setter method must exist. For example for the property ``provider`` a setter method
``setProvider`` must exist. The setter methods map the properties defined in the ``application.properties`` file to
private properties of the service implementation.

This is the only case a hivemind service is allowed to have private properties which are set by setter methods.
Otherwise a hivemind service must be stateless. All other properties must be declared ``private final`` and set by the
constructor.

The name of the setter methods is always ``setPropertyName`` (e.g. setSmsProvider).

.. code-block:: Java

   public class SmsParameterProviderImpl implements SmsParameterProvider {

       private String provider;
       private String username;
       private String password;
       private int warningLimit;
       private boolean simulateSms;

       public SmsParameterProviderImpl() {}

       @Override
       public void someMethodDefinedByTheInterface() {
         // ...
       }

       public void setProvider(String provider) {
           this.provider = provider;
       }

       public void setUsername(String username) {
           this.username = username;
       }

       public void setPassword(String password) {
           this.password = password;
       }

       public void setWarningLimit(int warningLimit) {
           this.warningLimit = warningLimit;
       }

       public void setSimulateSms(boolean simulateSms) {
           this.simulateSms = simulateSms;
       }
   }

In the ``application.properties`` file the properties can be set as follows:

.. code-block:: Properties

   sms.provider=websms
   sms.simulateSms=false
   sms.provider.username=USERNAME
   sms.provider.password=PASSWORD
   sms.warning.limit=10

Default Values
--------------

It is possible to set a default value. If the property is not set by any ``application.properties`` file the default value
is used. Default values can be defined as follows in the file ``hivemodule.xml``.

.. code-block:: XML

   <service-point id="SmsParameterProvider" interface="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProvider">
     <invoke-factory>
       <construct class="ch.tocco.nice2.optional.sms.impl.services.SmsParameterProviderImpl">
         <set property="provider" value="${sms.provider}"/>
       </construct>
     </invoke-factory>
   </service-point>

   <!-- Set the default value of the property sms.provider -->
   <contribution configuration-id="hivemind.FactoryDefaults">
     <default symbol="sms.provider" value="websms"/>
   </contribution>
