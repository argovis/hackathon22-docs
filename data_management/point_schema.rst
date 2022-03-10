.. _point_schema:

Point Schema
============

All data indexed by Argovis which can be cast as *point data* - in general,  any piece of data attached to a set of spacetime coodinates - are represented by the JSON schema described in this document. Argovis' point data schema is *hierarchical*,  in that all such schema inherit from the base point data schema; this creates two main advantages:

 - By requiring all point data encode their common features like coordinates,  IDs and other universal metadata identically,  it is trivial to include new data products in co-location APIs that search on these parametes.
 - Flexibility to represent arbitrary point data is maintained by allowing subject matter experts for a given measurement class to extend the base point data schema with additional schema that represent their specific instrument or measurement.

In this way,  we try to capture the flexibility of schema-less databases like mongodb,  while imposing just enough structure to capture the data validation opportunities afforded by strictly defined schema.

How to read this schema
-----------------------

Each entry in the schema fragments below contain a few keys:

- **type:** the primitive type,  format,  or object description of a valid entry for this field
- **description:** short comment on what this variable is
- fill value (optional): what this should be filled by if absent
- current vocabulary (optional): current set of possible values for this key,  with explanations as required.

Base point data schema
----------------------

This section describes the base point data schema from which all other point data in Argovis *must* inherit.

Required keys
+++++++++++++

All point data MUST include the following keys:

- ``_id``

  - **type:** string
  - **description:** a globally unique identifier for this record.

- ``basin``

  - **type:** int
  - **description:** integer index of basin. Can be provided by Argovis based on lat/lon in ``geolocation``.
  - **fill value:** -1,  used if reported lon/lat are on land.

- ``data_type``

  - **type:** string
  - **description:** token indicating the general class of data
  - **current vocabulary:** ``oceanicProfile``,  ``tropicalCyclone``

- ``date_updated_argovis``

  - **type:** ISO 8601 UTC datestring,  example ``1999-12-31T23:59:59Z``
  - **description:** time the record was added to Argovis

- ``geolocation``

  - **type:** geojson Point object
  - **description:** geojson Point tagging the lon/lat of this record.
  - **fill value:** ``{"type": "Point",  "coordinates": [0,  -90]}``

- ``source_info.source``

  - **type:** array of strings
  - **description:** data origin,  typically used to label major project subdivisions
  - **current vocabulary:** defined per project,  see schema extensions below.

- ``timestamp``

  - **type:** ISO 8601 UTC datestring,  example ``1999-12-31T23:59:59Z``
  - **description:** time the record measurement was made at.

Optional Keys
+++++++++++++

All point data MAY include the following keys; if meaningful data is available for any of these keys,  it should be represented in the format described.

- ``country``

  - **type:** string
  - **description:** ISO 3166-1 country code.

- ``data``

  - **type:** array of non-nested JSON documents
  - **description:** array indexes depth / altitude; individual documents are key/value pairs describing measurements made. Pressure or altitude must be present as one of the document keys. Example: ``{pres: 4.7,  psal: 33.987,  temp: 1.107}``
  - **current vocabulary:** defined per project,  see schema extensions below.

- ``data_center``

  - **type:** string
  - **description:** entity responsible for processing this record,  once received.
  - **current vocabulary:** defined per project,  see schema extensions below.

- ``data_keys`` **mandatory if** ``data`` **is present**

  - **type:** array of strings
  - **description:** a complete list of all the keys found in any document in the ``data`` object.

- ``data_warning``

  - **type:** array of strings
  - **description:** short string tokens indicating possible problems with this record.
  - **current vocabulary:**
    - ``missing_location``: one or both of longitude and latitude are missing
    - ``degenerate_levels``: data is reported twice for a given pressure / altitude level in a way that cannot be readily resolved.
    - ``missing_basin``: unable to determine meaningful basin code,  despite having a meaningful lat / lon (edge case in basins lookup grid)

- ``doi``

  - **type:** string
  - **description:** DOI for this record.

- ``instrument``

  - **type:** string
  - **description:** string token describing the device used to make this measurement,  like ``profiling_float``,  ``ship_ctd`` etc.
  - **current vocabulary:** TBD

- ``pi_name``

  - **type:** array of strings
  - **description:** name(s) of principle investigator(s)

- ``platform_id``

  - **type:** string
  - **description:** unique identifier for the platform or device responsible for making the measurements included in this record.

- ``platform_type``

  - **type:** string
  - **description:** make or model of the platform.
  - **current vocabulary:** TBD

- ``source_info.data_keys_source``

  - **type:** array of strings
  - **description:** list of measurement parameters as found in the source file

- ``source_info.date_updated_source``

  - **type:** ISO 8601 UTC datestring,  example ``1999-12-31T23:59:59Z``
  - **description:** date and time the upstream source file for this record was last modified

- ``source_info.source_url``

  - **type:** string
  - **description:** URL to download the original file from which the Argovis record was derived.

Argo profile schema extension
-----------------------------

All Argo data in Argovis is described as the union of the base point data schema and the following.

Base point schema vocabularies
++++++++++++++++++++++++++++++

The following keys from the base point schema have these vocabularies for Argovis:

- ``data`` keys: "bbp470", "bbp532", "bbp700", "bbp700_2", "cdom", "chla", "cndx", "cp660", "down_irradiance380", "down_irradiance412", "down_irradiance442", "down_irradiance443", "down_irradiance490", "down_irradiance555", "downwelling_par", "doxy", "molar_doxy", "nitrate", "ph_in_situ_total", "pres", "psal", "temp", "up_radiance412", "up_radiance443", "up_radiance490", "up_radiance555",  and the same again with "_qc" appended for the corresponding QC measurements.
- ``data_center``: TBD
- ``source_info.source``: ``argo_core``,  ``argo_bgc`` and ``argo_deep``

Required keys
+++++++++++++

- ``cycle_number``

  - **type:** int
  - **description:** probe cycle index

Optional keys
+++++++++++++

- ``data_keys_mode``

  - **type:** non-nested JSON document
  - **description:** JSON document with keys matching the entries of ``data_keys``,  and values indicating the variable's data mode
  - **current vocabulary:** ``R`` ealtime,  realtime ``A`` djusted,  or ``D`` elayed mode.

- ``fleetmonitoring``

  - **type:** string
  - **description:** URL for this float at https://fleetmonitoring.euro-argo.eu/float/

- ``geolocation_argoqc``

  - **type:** int
  - **description:** Argo's position QC flag

- ``oceanops``

  - **type:** string
  - **description:** URL for this float at https://www.ocean-ops.org/board/wa/Platform

- ``positioning_system``

  - **type:** string
  - **description:** positioning system for this float.
  - vocabulary: see Argo ref table 9

- ``profile_direction``

  - **type:** string
  - **description:** whether the profile was gathered as the float ascended or descended
  - **current vocabulary:** ``A`` scending or ``D`` escending.

- ``timestamp_argoqc``

  - **type:** int
  - **description:** Argo's date QC flag

- ``vertical_sampling_scheme``

  - **type:** string
  - **description:** sampling scheme for this profile.
  - **current vocabulary:** see Argo ref table 16

- ``wmo_inst_type``

  - tpye: string
  - **description:** instrument type as indexed by Argo.
  - **current vocabulary:** see Argo ref table 8

GO-SHIP profile schema extension
--------------------------------

All GO-SHIP data in Argovis is described as the union of the base point data schema and the following.

Base point schema vocabularies
++++++++++++++++++++++++++++++

The following keys from the base point schema have the following vocabularies for Argovis:

- ``data`` keys: TBD
- ``data_center``: TBD
- ``source_info.source``: TBD

Required keys
+++++++++++++

- ``expocode``

  - **type:** string
  - **description:** 

Optional keys
+++++++++++++

- ``cast``

  - **type:** int
  - **description:**

- ``cchdo_cruise_id``

  - **type:** int
  - **description:**

- ``station``

  - **type:** string
  - **description:**

- ``woce_lines``

  - **type:** array of strings
  - **description:**

