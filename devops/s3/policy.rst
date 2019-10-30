S3 Policy + ACL
===============

.. attention::

   You have to clone the ansible repository to access the files mentioned below. You can clone the project with the
   following command: **git clone ssh://${GERRIT_USERNAME}@git.tocco.ch:29418/ansible**

Show all options::

   ./utils/s3/s3policy.py --help


Apply the policies and acl's to a newly created bucket::

   git pull --rebase origin master # be sure to use the newest version!
   ./utils/s3/s3policy.py -b {BUCKET_NAME}


For more details see :doc:`/framework/architecture/s3/s3`.
