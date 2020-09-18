.. _initial-values:

#######################
Initial Value Generator
#######################

Country Initial Value Generator
===============================

This script generates the Tocco initial values for countries.

The data source is https://mledoze.github.io/countries

Execute Script
^^^^^^^^^^^^^^

**Prerequisite:** PyYAML is installed (``pip install PyYAML``)

1. Navigate to the country folder (``cd src/bin/initialvalues/country``)
2. Go to the data source on GitHub and download the latest version of the ``countries.json`` file
3. Copy the ``countries.json`` file to the ``src/bin/initialvalues/country/input`` directory
4. Run the script with ``python3 CountryImporter.py``
5. Copy the existing initial value file (from ``optional/address/module/db/initialvalues/Country.yaml``) to ``src/bin/initialvalues/input/Country.yaml``
6. Run the script with ``python3 CountryDiff.py``
7. For each country code in the *Country code no longer exists* list do the following (change set location should be ``optional/address/module/db/initialvalues/2.xx_update_country.xml``):

    * If a country still exist with another iso3 code (new iso3 code must be listed in *New country code added*), then add the following change set:

    .. code-block:: XML

        <changeSet author="anonymous" dbms="postgresql" id="ID" runOnChange="true">
            <preConditions onFail="MARK_RAN">
               <tableExists tableName="nice_country"/>
            </preConditions>
            <update tableName="nice_country">
               <column name="iso3" value="NEW_VALUE"/>
               <where>iso3 = 'OLD_VALUE'</where>
            </update>
        </changeSet>


   Set ``ID`` to something like ``updateCountry_XKV_to_UNK/2.27``, ``OLD_VALUE`` is the old iso3 code and ``NEW_VALUE`` is the new iso3 code

   Additionally, update ``src/bin/initialvalues/input/{licence_plate, sorting, zip_city}.csv`` if necessary

    * Else, write a change set to set the country to inactive:

    .. code-block:: XML

        <changeSet author="anonymous" dbms="postgresql" id="ID" runOnChange="true">
            <preConditions onFail="MARK_RAN">
              <tableExists tableName="nice_country"/>
            </preConditions>
            <update tableName="nice_country">
              <column name="active" value="false"/>
              <where>iso3 = 'ISO_CODE'</where>
            </update>
        </changeSet>

    Set ``ID`` to something like ``updateCountry_remove_ANT/2.27`` and ``ISO_CODE`` is the iso3 code

8. Copy the output files (in ``src/bin/initialvalues/country/output``) to the Tocco project:

    * Override the existing ``Country.yaml`` file in ``optional/address/module/db/initialvalues``
    * Override the country name section in the language file ``language_XX.properties`` which is located at ``optional/address/module/model/textresources``
9. Check the git diff to verify the data source quality

Additional Information
^^^^^^^^^^^^^^^^^^^^^^

* The ``licence_plate`` field is not part of the data source. There is a static file under ``src/bin/initialvalues/country/input/licence_plate.csv`` as data source in the format ``Iso3,Licence_plate_code``
* The ``zip_city`` field is an internal Tocco field. All countries which are listed in the ``src/bin/initialvalues/country/input/zip_city.csv`` file are set to ``true`` otherwise the value will be ``false``
* The ``sorting`` field is an internal Tocco field. Per default a country obtains the value ``100``. If a non-default value is required there is a static file under ``src/bin/initialvalues/country/input/sorting.csv`` in the format ``Iso3,number`` to define the value per country
* Some countries have multiple currency codes and calling codes. In such a case the values are comma-separated written into the text field