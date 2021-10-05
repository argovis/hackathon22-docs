Profiles API Alpha Product
==========================

An alpha demo of a new ``/profiles`` endpoint is available at TBD.

Project Goals
-------------

 - We hope to design and build an API that is intuitive for users and easy to maintain for the Argovis team. Currently, many ``/catalog`` and ``/selection`` API endpoints return very similar profile-like data, with differences that aren't always obvious from the route names. We hope that a ``/profiles`` route with a single ``GET`` request method will provide an obvious place for consumers to pull profile data, and serve many if not all of the design intents of the ``/catalog`` and ``/selection`` routes. 
 - Looking to the future, we hope to offer an API tier that spans all profile-like data, whether it comes from Argo, go-ship, or other endeavors.
 - The goal of this alpha demo is to gather feedback on design decisions as development continues.

Usage
-----

All requests should currently be made as ``GET`` requets to the ``/profiles`` endpoint. Reducing the amount of data returned can be achieved with the following query string parameters; using multiple query string parameters ``AND``s the corresponding requirements.

Mandatory Query String Parameters
+++++++++++++++++++++++++++++++++

 - ``startDate`` (MANDATORY): formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``. Profiles will be returned if they are timestamped at or after this time.
 - ``endDate`` (MANDATORY): formatted as ISO 8601 UTC: ``1999-12-31T23:59:59Z``. Profiles will be returned if they are timestamped before this time.

.. admonition:: Why are dates mandatory?

   The Argovis API paginates data by enforcing a *maxium of 3 months* of data returned in any one query, in order to implement pagination in a way that is natural and easy to understand for non-experts. See our examples (link TBD) for code to loop over this date-based pagination.

Optional Query String Parameters
++++++++++++++++++++++++++++++++

 - ``polygon``: formatted as ``[[lon,lat],[lon,lat],[lon,lat]...]``; note the first and last ``[lon,lat]`` pairs must be identical. Profiles will be returned if they fall within the closed polygon created by connecting the coordinate pairs in the order they are serialized.
 - ``box``: formatted as ``[[lower left lon,lower left lat],[upper right lon,upper right lat]]``. Profiles will be returned if they fall within the box defined by its lower left and upper right coordinate corners.
 - ``ids``: formatted as ``id_1,id_2,id_3,...``. Profiles will be returned if they have an ID in the list provided.
 - ``platforms``: formatted as ``platform_1,platform_2,platform_3,...``. Profiles will be returned if they have a platform number in the list provided.
 - ``presRange``: formatted as ``minPres,maxPres``. Only levels with ``measurements.pres``  that fall within the range bracketed by ``minPres`` and ``maxPres`` will be returned.
 - ``coreMeasurements``: formatted as ``meas_1,meas_2,...``. Specifies the core QC'ed measurements that will be returned as part of the ``measurements`` key. Notes:
     - Valid choices: ``pres``, ``temp``, ``psal``, ``all``
     - ``pres``sure is always returned, no matter what.
     - ``all`` returns all available measurements, and is equivalent to ``coreMeasurements=temp,psal``. 
 - ``bgcMeasurements``: formatted as ``meas_1,meas_2,...``. Specifies the biogeochemical measurements that will be returned as part of the ``bgcMeas`` key. Notes:
     - Valid choices: ``pres``,``temp``,``psal``,``cndx``,``doxy``,``chla``,``cdom``,``nitrate``,``bbp700``,``down_irradiance412``,``down_irradiance442``,``down_irradiance490``,``downwelling_par``, ``all``
     - ``pres``sure and ``pres_qc`` are always returned, no matter what.
     - Requesting a variable also returns its QC (ie requesting ``doxy`` will also return ``doxy_qc``).
     - ``all`` returns all available measurements and their QC.

.. admonition:: Data or Metadata?

   If your query string includes neither ``coreMeasurements`` or ``bgcMeasurements``, the request will return profile metadata only. If your query string includes either of those and a matching profile has no levels with the appropriate measurements after filtering, it will be dropped from the returned results.


Examples
--------

Mapping to old endpoints
------------------------

Outstanding Issues
------------------

Key Standardization
+++++++++++++++++++

Argo and goship data have similar but not identical names for some keys. Ideally, we would have a hierarchical schema for profile data:

 - *Common Mandatory Parameters* are parameters that every profile from every source must have. Examples are ``lat`` and ``lon``.
 - *Common Optional Parameters* are parameters that may or may not be included in a profile, but should have consistent naming and meaning across sources. An example is ``bgcMeas``.
 - *Origin-specific Parameters* are parameters unique to a data origin, like Argo or go-ship.


