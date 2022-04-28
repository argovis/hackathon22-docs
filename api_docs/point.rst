Profiles API Beta Product
=========================

Argovis' ``/profiles`` API endpoint is designed to support expressive searches of profile data that allow users to download only and exactly the data they need. This doc describes the ``/profiles`` endpoint and its options, along with some other related endpoints for exploring oceanic profiles.

Usage
-----

Unless otherwise noted, all routes support only ``GET`` requests at this time.

.. admonition:: Longitudes and Latitudes

   For all queries on all endpoints, where longitudes and latitudes are supported, they must fall on [-180,180] and [-90,90], respectively.

/profiles
+++++++++

The base ``/profiles`` endpoint is the place to go for all profile data. Reducing the amount of data returned can be achieved with the following query string parameters; using multiple query string parameters ``AND`` s the corresponding requirements. All parameters are optional unless otherwise noted. See :ref:`point_schema` for the possible vocabularies of the parameters filtered on, and more details on the objects returned.

 - ``startDate`` 

   - formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``.
   - Filters against point schema property: ``timestamp``
   - description: Profiles will be returned if their ``timestamp`` is at or after this time.

 - ``endDate``

   - formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``. 
   - Filters against point schema property: ``timestamp``
   - description: Profiles will be returned if their ``timestamp`` is before this time.

 - ``polygon``

   - formatted as ``[[lon,lat],[lon,lat],[lon,lat]...]``; note the first and last ``[lon,lat]`` pairs must be identical.
   - Filters against point schema property: ``geolocation``
   - description: Profiles will be returned if their ``geolocation`` property falls within the closed polygon created by connecting the coordinate pairs in the order they are serialized.

 - ``box``

   - formatted as ``[[lower left lon,lower left lat],[upper right lon,upper right lat]]``. 
   - Filters against point schema property: ``geolocation``
   - description: Profiles will be returned if their ``geolocation`` falls within the box defined by its lower left and upper right coordinate corners.

 - ``center`` plus ``radius``

   - formatted as ``center lon, center lat`` and a distance in km, respectively. 
   - Filters against point schema property: ``geolocation``
   - description: Profiles will be returned if their ``geolocation`` falls within the circle defined by this pair of parameters.

 - ``id``

   - formatted as string. 
   - Filters against point schema property: ``_id``
   - description: Return only the profile with ``_id`` property matching this unique ID.

 - ``platform``

   - formatted as string
   - Filters against point schema property: ``platform_id``
   - desciption: Profiles will be returned if their ``platform_id`` matches this value.

 - ``presRange``

   - formatted as ``minPres,maxPres``. 
   - Filters against point schema property: ``data.pres``
   - description: Only levels with pressure that fall within the range bracketed by ``minPres`` and ``maxPres`` will be returned.

 - ``dac``

   - formatted as string. 
   - Filters against point schema property: ``data_center``
   - description: Only returns profiles with ``data_center`` matching this value.

 - ``source``

   - formatted as a comma delimited strings
   - Filters against point schema property: ``source_info.source``
   - description: return profiles that satisfy all listed source strings. Also accepts negation like ``~argo_bgc``, meaning 'not matching ``argo_bgc``'.

 - ``woceline``

   - formatted as string. 
   - Filters against point schema property: ``woce_lines``
   - description: Profiles from the indicated GO-SHIP woce line will be returned.

 - ``data``

   - formatted as comma delimited list of strings. 
   - Filters against point schema property: ``data`` and its subkeys
   - Description: return profiles that have all the specified data variables present in their ``data`` key. Also accepts negations, like ``data=pres,~doxy`` would be profiles that have pressure but not dissolved oxygen. Also accepts ``metadata-only`` in the list of strings, which will cause only the metadata of the matched profiles to be returned.

 - ``compression``

   - description: if set to any value, a minified ``data`` array will be returned: instead of a dictionary for each level, ``data`` will be a list of lists, where each inner list contains the values requested for a level, in the same order as the ``data_keys`` object on the corresponding returned profile.

.. admonition:: Required Parameters and Response Sizes

   In order to constrain the amount of data returned in a single request, your query must either: 

    - Include ``startDate`` and ``endDate`` and return less than 1000 profiles. If it's too broad, you'll recieve an HTTP/400 error and should consider cutting scope to a specific region, date range, or other filter. Bear in mind that you can always make more requests to cover more cases!
    - Include ``id`` to get a single profile.

.. admonition:: Data or Metadata?

   If your query string does not include ``data``, the request will return profile metadata only. If your query string includes either of those and a matching profile has no levels with the appropriate measurements after filtering, it will be dropped from the returned results.

There are also two subroutes under ``/profiles``:

 - ``/profiles/listID``: takes the same query string as ``/profiles``, except ``id`` is not accepted as a query parameter (since it's not useful to search for IDs by ID). Furthermore, up to 10k profile IDs may be returned by this route.
 - ``/profiles/overview``: take no query string parameters, and returns some high level summary data of the profiles available, like what DACs are represented, how many Argo BGC and Deep profiles are present, and some others.

/platforms
++++++++++

There are two simple `/platforms` routes, which summarize some profile information grouped by platform ID:

 - ``/platforms?platform=<platform ID>`` returns metadata for the platform specified.
 - ``/platforms/mostRecent?platforms=<comma separated list of platform IDs>`` returns some information about the listed platform's most recent location and measurements.

 



