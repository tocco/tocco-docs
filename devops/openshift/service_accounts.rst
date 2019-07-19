Service Accounts
================

Beside the normal user accounts that are used to login interactively with the openshift client, openshift provides service accounts.
With a service accounts you can login interactively as well, but as the name says, this accounts are rather used by services than people.

What Do We Need Service Accounts For?
-------------------------------------

When we do a deployment with teamcity we have to run certain openshift commands like: 'oc get' or  'oc rollout'.
These commands have to be executed by an account. That no one has to be logged in all the time with his own account, we use service accounts.
With this type of account we just have to login once and the session will stay.

How To Create A Service Account
-------------------------------

In openshift you can only create service accounts bound to a project and not to the whole cluster. First this seems to be a little bit weird, and yes it is.
So it is common sense to create on project just for service accounts. You can easily give permissions for other projects to the service account. 
The syntax to create an account is very easy, but be sure that you are in the right project.

.. parsed-literal::
    
   $ oc create sa teamcity

   $ oc get sa

     NAME       SECRETS   AGE

     builder    2         57d
     default    2         57d
     deployer   2         57d
     teamcity   2         2S

How to add a service account to a project see :ref:`add-sa-reference-label`.

Login With A Service Account
----------------------------

The login with a service account isn't the common style with username and password. The credentials are replaced with a token.
So you only need the token to login. The challenge here is to find the token, but don't worry we documented it for you.

.. parsed-literal::

   $ oc project toco-serviceaccounts


.. parsed-literal::

   $ oc ge sa

   $ oc describe sa teamcity

   Name:           teamcity
   Namespace:      toco-serviceaccoutns
   Labels:         <none>
   Annotations:    <none>

   Image pull secrets:     teamcity-dockercfg-9mvt0

   Mountable secrets:      **teamcity-token-fz73r**
                           teamcity-dockercfg-9mvt0

   Tokens:                 **teamcity-token-fz73r**
                           teamcity-token-k25qk

.. parsed-literal::

   $ oc describe secret **teamcity-token-fz73r**

   Name:           teamcity-token-fz73r
   Namespace:      toco-serviceaccoutns
   Labels:         <none>
   Annotations:    kubernetes.io/service-account.name=teamcity
                   kubernetes.io/service-account.uid=65638a57-c537-11e7-862d-fa163ec9e279

   Type:   kubernetes.io/service-account-token

   Data
   ====
   namespace:      20 bytes
   service-ca.crt: 2235 bytes
   token:          **token** 
   ca.crt:         1066 bytes


.. parsed-literal::

   oc login --token= **token**
