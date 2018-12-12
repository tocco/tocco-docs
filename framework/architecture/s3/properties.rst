Properties file
===============

Introduction
------------

The S3 module is available from version 2.19.

Connect to database over ssh
----------------------------

The S3 storage is accessible publicly.
But if you change the "dataSource.serverName" in the "hikaricp.properties" to localhost you have to adjust the "s3.checksum" property.
The checksum for use with localhost is already in the s3.properties file but commented out. It just needs to be commented in.
