Grids API Beta Product
======================

Argovis' ``/grids`` API endpoint is designed to support expressive searches of grid data that allow users to download only and exactly the data they need. This doc describes the ``/grids`` endpoint and its options.

Demo Notebook
-------------

You can see the ``/grids`` endpoint in action in our demo notebook, available on `Binder <https://github.com/argovis/demo_notebooks>`_.

Usage
-----

Unless otherwise noted, all routes support only ``GET`` requests at this time.

.. admonition:: Longitudes and Latitudes

   For all queries on all endpoints, where longitudes and latitudes are supported, they must fall on [-180,180] and [-90,90], respectively.

/grids
++++++

The base ``/grids`` endpoint is the place to go for all gridded data; at this time Argovis supports the Roemmich-Gilson Argo climatology grids, and an ocena heat content grid. Reducing the amount of data returned can be achieved with the following query string parameters; using multiple query string parameters ``AND`` s the corresponding requirements. All parameters are optional unless otherwise noted. See :ref:`grid_schema` for the possible vocabularies of the parameters filtered on, and more details on the objects returned.

 - ``gridName`` (mandatory)

  - formatted as a single string
  - description: specifies the grid to query, currently one of ``rgTempTotal``, ``rgPsalTotal``, or ``ohc``.

 - ``startDate`` (mandatory)

   - formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``.
   - Filters against grid schema property: ``t``
   - description: Columns of grid points will be returned if their ``t`` is at or after this time.

 - ``endDate`` (mandatory)

   - formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``. 
   - Filters against grid schema property: ``t``
   - description: Columns of grid points will be returned if their ``t`` is before this time.

 - ``polygon`` (mandatory)

   - formatted as ``[[lon,lat],[lon,lat],[lon,lat]...]``; note the first and last ``[lon,lat]`` pairs must be identical.
   - Filters against grid schema property: ``g``
   - description: Columns of grid points will be returned if their ``g`` property falls within the closed polygon created by connecting the coordinate pairs in the order they are serialized.

 - ``presRange``

   - formatted as ``min pres,max pres``
   - description: filters out grid levels that fall outside the specified range.