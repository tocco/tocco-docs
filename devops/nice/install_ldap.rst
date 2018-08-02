How to get LDAP running on Openshfit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

What is LDAP used for?
======================

LDAP is used to expose certain directories from a Database to the outer world. In our case for example we expose credentials, so that you have the same login everywhere.

Install LDAP on nice2
=====================

Create the Image with the Key
-----------------------------

.. attention::

   You have to clone the ansible repository to access the files mentioned below. You can clone the project with the
   following command: **git clone ssh://${GERRIT_USERNAME}@git.tocco.ch:29418/ansible**

1. Copy the key into the ldap-image diretory in the ansible project. It has to be an SSL Private key in a Java keystore.

.. code::

   cp  ${PATH_TO_KEYFILE}/key.ks ${PATH_TO_ANSIBLE}/openshift/ldap-image

2. Set the right path for the key file inside the Dockerfile. Inside the Dockerfile the path of the key file is denoted as ${PATH_TO_KEYFILE}, so just execute the following

.. code::

   export PATHTOKEYFILE="/persists/${KEYFILE}"

3. Build the docker image and push it.

.. code::

   docker built -t registry.appuio.ch/toco-nice-${INSTALLATION}/${CUSTOMER}:ldap .

   docker push registry.appuio.ch/toco-nice-${INSTALLATION}/${CUSTOMER}:ldap

Add the Secret and Service to the Project
-----------------------------------------

1. Create a secret for to mount the key into the nice container. There is a yaml file as template, just execute it with oc process.

.. code::

   oc process -f ldap-secret-template.yml SECRET_DISK_SPACE=1Gi CUSTOMER=${CUSTOMER} PASSWORD=${SECRET_PASSWORD}| oc create -f -

2. Create a service for the LDAP-Server. This can also be done with a yaml Template.

.. code::

   oc process -f ldap-service-template.yml | oc create -f -

Adjust the Deployment Config
----------------------------

1. Add the following APP Parameters to the Deployment Config.

.. code::

   oc set env dc/nice NICE2_APP_nice2__optional__ldapserver__enabled="true" NICE2_APP_nice2__optional__ldapserver__port="10389" NICE2_APP_nice2__optional__ldapserver__certificatePassword="${CERTIFICATE_PASSWORD}" NICE2_APP_nice2__optional__ldapserver__keyStoreFile="/persist/${KEYFILE}"

2. Mount the Secret into the nice Container

.. parsed-literal::

   oc volume dc/nice -m /persist --secret-name ldap-secret --claim-name ldap-secret --add
