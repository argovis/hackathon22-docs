.. _grid_schema:

Grid Schema
===========

This document describes the schema to which gridded Earth system data is cast in Argovis.

How to read this schema
-----------------------

Each entry in the schema fragments below contain a few keys:

- **type:** the primitive type, format,  or object description of a valid entry for this field
- **description:** short comment on what this variable is
- **fill value** (optional): what this should be filled by if absent
- **current vocabulary** (optional): current set of possible values for this key,  with explanations as required.

Schema enforcement & population
-------------------------------

All these schema are enforced via mongodb's built-in schema validation; those schema validation rules are defined in the ``grids.py`` and ``grids-meta.py`` scripts at https://github.com/argovis/db-schema.

Once empty collections are generated by the script above, it is populated by pipelines that translate from the formats of upstream data providers, to mongodb-appropriate JSON that matches these schemas. Each section below contains links to the code backing these pipelines. Note that each gridded product lives in its own mongo collection at this time.

Grid Metadata Schema
--------------------

In order to avoid repetition of information that is the same for all grid points, each grid in collection ``<grid name>`` will have a record in collection ``grids-meta`` with ``_id=<grid name>`` that carries all the metadata for that grid. The schema of ``grids-meta`` documents are described in this section.

Scripts for translating upstream data to these schema:

 - Roemmich-Gilson temperature and salinity: https://github.com/argovis/grid-sync/blob/main/translate-rg-grid.py

Required keys
+++++++++++++

*Note all keys are required for metadata docs*.

- ``_id``

  - **type:** string
  - **description:** a string that matches the collection name that contains the data that should be associated with this metadata record
  - **current vocabulary**: ``rgTempTotal``, ``rgPsalTotal``, ``ohc``

- ``units``

  - **type:** string
  - **description:** the units of measure by which the ``d`` variables in the corresponding grids should be interpreted.

- ``levels``

  - **type:** array of floats
  - **description:** the pressures in dbar of all levels of this grid, in the same order as the ``d`` record of the corresponding grid data.

- ``date_added``

  - **type:** ISO 8601 UTC datestring,  example ``1999-12-31T23:59:59Z``
  - **description:** time the record was added to Argovis

- ``lonrange``

  - **type:** array of floats
  - **description**: minimum and maximum longitudes, in degrees, found in this grid

- ``latrange``

  - **type:** array of floats
  - **description:** minimum and maximum latitudes, in degrees, found in this grid

- ``timerange``

  - **type:** array of ISO 8601 UTC datestring
  - **descriptionL** minimum and maximum dates found in this grid

- ``loncell``

  - **type:** float
  - **description:** grid spacing in degrees in longitude

- ``latcell``

  - **type:** float
  - **description:** grid spacing in degrees in latitude

Grid Data Schema
----------------

Each gridded product is stored in its own collection, the name of which has a corresponding entry in the ``grids-meta`` collection. Documents in the gridded data collections obey the following schema, which describe a profile-like column of a grid.

Scripts for translating upstream data to these schema:

 - Roemmich-Gilson temperature and salinity: https://github.com/argovis/grid-sync/blob/main/translate-rg-grid.py

Required keys
+++++++++++++

*Note all keys are required for gridded docs*.

- ``g``

  - **type:** geojson Point object
  - **description:** geojson Point tagging the lon/lat of this record.

- ``t``

  - **type:** ISO 8601 UTC datestring,  example ``1999-12-31T23:59:59Z``
  - **description:** time this data corresponds to.

- ``d``

  - **type:** array of floats
  - **description:** the variable data for this column of the grid. These entries correspond in order to the ``levels`` array in the corresponding metadata document which describes their depth, and carry the units of the ``units`` entry in the same metadata document.