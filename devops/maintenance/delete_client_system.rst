Delete Tocco client-system
^^^^^^^^^^^^^^^^^^^^^^^^^^

Delete Client-System on "Nine.ch"
=================================

1. Remove monitoring.

2. Remove in the mgrctl the part of client.

3. Test config with mgrctl restart.

3. Remove DNS.

5. Rename the databases ([client], [client]test, [client]_history and [client]_test_history) on the databaseserver to del_nice_CLIENTNAME

6. Wait at least one day for the automatic backup to run.

7. Now you can access the rename databases `del_...` delete.

8. Remove the project project of Client in Teamcity.

9. Update status to "veraltet" in the Tocco BackOffice.

10. Set the customer module that is linked to the installation to "Obsolete

11. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)



Delete Client-System on "VSHN"
==============================

1. Remove monitoring

3. Remove DNS

2. Cloudscale: scale the project to be deleted to 0 instanzes.

3. Git.vshn.net: Go "Repository" and click "Files".

4. Remove the code part of client conf for the database from the "Puppet-Config"

5. Rename the databases ([client], [client]test, [client]_history and [client]_test_history) on the databaseserver to del_nice_CLIENTNAME

6. Wait at least one day for the automatic backup to run.

7. Now you can access the rename databases `del_....` delete.

8. Remove the project project of Client in Teamcity.

9. Update status to "veraltet" in the Tocco BackOffice.

10. Set the customer module that is linked to the installation to "Obsolete

11. Delete the Customer's Maven module from the Nice2-Git repository (if there are no other installations that require the Customer module)

