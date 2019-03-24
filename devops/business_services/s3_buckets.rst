S3 Buckets
==========

To create buckets in S3, you need an Access to access the Web client.
With the following URl: the client can be called: ``https://control.cloudscale.ch/objects``


.. figure:: resources/s3_cloudscale_webclient.png

Create new S3 user
------------------

Press the Button on top-right ``Create a new Object User``
Add a locical username.

.. figure:: resources/create_new_user.png

A new user can easily be created via curl.
This is especially useful when many users need to be created.


.. parsed-literal::

        curl -i -H 'Authorization: Bearer [my access token]' -F display_name=[new username] https://api.cloudscale.ch/v1/objects-users;

Check the access-key and security-Key
-------------------------------------

.. figure:: resources/check_keys.png

Create new S3-Bucket
--------------------

presss the button ``add a bucket`` and insert a logical bucketname.

.. figure:: resources/add_bucket.png

