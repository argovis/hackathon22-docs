Restore Mongo from Backup
=========================

In the event of a database failure or migration, we may need to restore MongoDB databases and collections from backups. In this tutorial, you'll learn how to do so; we assume you have a running version of Argovis before you begin.

1. First, we need to figure out where to put our backed up data so mongo can read it back in. Start by listing your containers; you should see something like the example output below:

   .. code:: bash

      ~ $ docker container ls

      CONTAINER ID   IMAGE                      ...
      6d9e71e4d6db   argovis/ng:dev.            ...
      feeac652a257   mongo:4.2.3                ...

You're interested in the container ID of the container using the ``mongo`` image, ``feeac652a257`` in my example (yours will be different).

2. Use the container ID you found to inspect that container and determine its filesystem mount points; you're looking for the ``Mounts`` block in the output generated:

   .. code:: bash

      ~ $ docker container inspect <mongo container ID>

        "Mounts": [
            {
                "Type": "volume",
                "Name": "1946b3562a81...",
                "Source": "/var/lib/docker/volumes/1946b3562a81.../_data",
                "Destination": "/data/configdb",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "bind",
                "Source": "/home/ubuntu/argovis-tutorial/data/argovis/storage/mongodb",
                "Destination": "/data/db",
                "Mode": "Z",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

You're interested in the ``Source`` directory associated with the ``"Destination": "/data/db"``. That ``Source`` location is the directory on your machine where you need to place your mongo collection backup, in my example ``/home/ubuntu/argovis-tutorial/data/argovis/storage/mongodb``.

3. Change to the ``Source`` path you found above, and make a subdirectory to hold your backup bson and metadata; I'll call this subdirectory ``goship``, but you can name yours as appropriate:

   .. code:: bash

      ~ $ cd /home/ubuntu/argovis-tutorial/data/argovis/storage/mongodb

      mongodb $ mkdir goship

Place your mongo collection backup bson and metadata in that subdirectory; mine are called ``profiles.bson`` and ``profiles.metadata.json``, but it can be called anything you like.

3. Tell mongo to re-load your collection from backup. Make sure to insert the mongo container ID you found above, modify the ``goship`` part of the path to match the subdirectory you made above, and feel free to change the ``--db`` and ``--collection`` flags to name your database and collections anything you like.

   .. code:: bash

      ~ $ docker container exec <mongo container ID> \
          mongorestore --batchSize=64 \
          --db=goship --collection=profiles \
          /data/db/goship/profiles.bson

.. admonition:: Warning

   Collection restoration is a memory intensive process! While the restoration proceeds, make sure to monitor memory usage and ensure it doesn't exceed what's available. If so, kill the restore and decrease the ``--batchSize`` parameter to reduce the amount of memory allocated by mongo on restore.

4. Validate that your database was restored by creating an interactive connection to your mongodb and listing your databases:

   .. code:: bash

      ~ $ docker container exec  -it <mongo container ID> mongo

      > show dbs

      admin   0.000GB
      argo    0.003GB
      config  0.000GB
      goship  2.464GB
      local   0.000GB

      > exit

I can see my ``goship`` database is present and has content, so this looks good. Before exiting, feel free to explore some more to make sure your data has been restored in the way you expect.

5. After re-loading data, your container might have allocated a large amount of memory during restore.

   .. code:: bash

      ~ $ docker container stats --no-stream

      CONTAINER ID   NAME                       CPU %     MEM USAGE / LIMIT     MEM % ...
      cf17e99f65c5   argovisng_ng_1             0.27%     67.81MiB / 31.36GiB   0.21% ...
      c5f477eb4a83   argovisng_database_1       0.24%     12GiB / 31.36GiB      38.28%...

In this example, after I restored my goship database, the database container has allocated a whopping 12 GB of memory. Restart the container to flush memory:

   .. code:: bash

      ~ $ docker container restart <database container ID>

Check your container stats again and your database should have freed up most of the memory it was consuming.



