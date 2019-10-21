Connect to Locally Running Nice2 to Production/Test DB/S3
=========================================================

#. Get the setting from OpenShift:

   .. parsed-literal::

      $ oc set env -c nice dc/nice --list \|grep -P '^NICE2_(HIKARI\|S3)_'
      NICE2_HIKARI_minimumIdle=2
      **NICE2_S3_s3.main.endpoint=https://objects.cloudscale.ch**
      **NICE2_S3_s3.main.bucketName=tocco-nice-toccotest**
      **NICE2_S3_s3.main.accessKeyId=WGRDSJ4J3J5TBNEX1B7T**
      **NICE2_S3_s3.main.secretAccessKey=OgymXsYjMQdGSEcYiRYFCWc6x6iMwUVXmKP5PLT4**
      NICE2_HIKARI_maximumPoolSize=5
      NICE2_HIKARI_leakDetectionThreshold=30000
      **NICE2_HIKARI_dataSource.databaseName=nice_toccotest**
      **NICE2_HIKARI_dataSource.password=C6HAgYDIVZDyzvTHLpXjShiB**
      NICE2_HIKARI_dataSource.serverName=db1.tocco.cust.vshn.net
      **NICE2_HIKARI_dataSource.user=nice_toccotest**

#. Create a local override file for the database connection settings.

   Create or edit the file ``customer/${CUSTOMER}/etc/hikaricp.local.properties``:

   .. parsed-literal::

      dataSource.serverName=localhost  # always localhost

      # Use setting from previous output (without the NICE2_HIKARI\_ prefix)
      **dataSource.user=nice_toccotest**
      **dataSource.password=C6HAgYDIVZDyzvTHLpXjShiB**
      **dataSource.databaseName=nice_toccotest**

#. Create a local override file for the S3 connection settings.

   Create or edit the file ``customer/${CUSTOMER}/etc/s3.local.properties``:

   .. parsed-literal::

       # Configuration from previous output (without the NICE2_S3\_ prefix)
       **s3.main.endpoint=https://objects.cloudscale.ch**
       **s3.main.bucketName=tocco-nice-tocco**

       # This two entries must exist and be empty to override the default overlay
       # bucket configured in s3.properties.
       s3.overlay.accessKeyId=
       s3.overlay.secretAccessKey=

#. Create an SSH tunnel to the DB server.

   .. code::

       ssh db1.tocco.cust.vshn.net -L 5432:localhost:5432 -N

   This will forward TCP port 5432, where Postgres is running, to your local
   machine. Thus, as configured above, you can now connect to the DB server
   via *localhost:5432*.

#. Start Nice2
