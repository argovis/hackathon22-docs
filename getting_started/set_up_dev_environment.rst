.. _setup_argovis:

Setting Up Argovis
==================

In this tutorial, you'll learn how to deploy a toy version of Argovis to your laptop or development environment, so you can experiment with how it works and try modifying the codebase. In order to follow this tutorial, you'll need:

- A linux or OSX environment with:
   - a bash shell
   - docker
   - docker-compose
   - git

Quickstart Setup
----------------

1. Make a directory to store all your Argovis assets. We'll put this in our home directory, but you can put it anywhere you like:

   .. code:: bash

      ~$ mkdir ~/argovis

      ~$ cd ~/argovis

2. Make a directory to hold your database files:

   .. code:: bash

      argovis $ mkdir data

You'll need the absoute path to this directory in a later step; find it and note it down now before returning to the ``~/argovis`` directory:

   .. code:: bash

      argovis $ cd data

      data $ pwd
      
      data $ cd ..

For example, my absolute path returned by ``pwd`` was ``/Users/billmills/argovis/data``.

3. Download the minimal code necessary to start up Argovis:

   .. code:: bash

      argovis $ git clone -b main https://github.com/argovis/argovisNg

      argovis $ cd argovisNg

4. Before we launch the app, we need to tell it where to look for the data directory you made above. In your text editor of choice, open up ``docker-compose.yml`` and look for the lines describing the database volume mount, near the middle, which look like:

   .. code:: bash

      volumes:
        - /storage/mongodb:/data/db:Z # development
        #- /data/argovis/storage/mongodb:/data/db:Z # production

On the line commented as ``# development``, change ``/storage/mongodb`` to the absolute path to your ``data`` directory you found above, in my case ``/Users/billmills/argovis/data``.

5. Start Argovis:

   .. code:: bash

      argovisNg $ docker-compose up -d

After a few seconds, visit ``http://localhost:3000`` in your web browser; you should see your very own local version of Argovis.

6. Once you're satisfied all is well, tear your local deployment down:

   .. code:: bash

      argovisNg $ docker-compose down

.. _startup_load_test_data:

Loading Data
------------

The version of Argovis you have up and running from the previous section works, but has no actual data loaded in it. In this section we'll see how to fetch and load some Argo float data so there's something to look at in our development environment.

The FTP server at ftp://ftp.ifremer.fr/ifremer/argo contains all the Argo profile data used by the database. Getting the data into the database involves downloading files and directories to argo. We will use the rsync utility to download a few profiles, and use the argo-database repository to convert these files to entries on the server.

1. Start your development version of argovis back up if you tore it down at the end of the last section:

   .. code:: bash

      ~$ cd ~/argovis/argovisNg

      argovisNg $ docker-compose up -d

2. Return to the directory containing all your Argovis assets, and clone the Argovis database management code:

   .. code:: bash

      ~$ cd ~/argovis

      argovis $ git clone -b dev https://github.com/argovis/argo-database
      
      argovis $ cd argo-database


3. Make a directory to hold some demo profiles:

   .. code:: bash

      argo-database $ mkdir -p profiles/aoml

4. Use ``rsync`` to fetch some profiles:

   .. code:: bash

      argo-database $ rsync -arvzhim --delete \
          --include='**/' \
          --include='**/profiles/[RDM]*.nc' \
          --exclude='*' \
          --exclude='**/profiles/B*' \
          vdmzrs.ifremer.fr::argo/aoml/4902911 ./profiles/aoml

All the profiles from the float 4902911 should have been written to ``argo-database/profiles/aoml/4902911/profiles``. You can add more or all of the floats in this fashion if you wish.

5. In order to simplify environment setup, we'll run our database update script from a containerized environment that has all the tooling you'll need pre-installed. Start and connect to this contianerized environment from the ``argo-database`` directory:

   .. code:: bash

      argo-database $ docker container run \
          -u $(id -u):$(id -g) \
          -it --rm --net=host \
          -v $(pwd):/usr/src/argo-database/:Z \
          argovis/argo-db:tut bash

6. Move to the appropriate directory in the container's filesystem, run the data upload script, and exit the contianerized environment when complete:

   .. code:: bash

      argo-database $ cd add-profiles

      add-profiles $ python add_profiles.py \
          --mirrorDir /usr/src/argo-database/profiles \
          --logName tutorial.log

      add-profiles $ exit

7. In your browser, visit your deployment of Argovis at ``http://localhost:3000``, and use the search box in the left sidebar to search for float 4902911; if all is well, you should see this float's data populated on the map.

.. _building_dev_versions:

Building Dev Versions
---------------------

So far, we've relied on pre-built container images for Argovis to get things stood up and working quickly. But, of course you'll want to build your code into new containers and try them out as part of your local deployment. We'll learn how to do that in this section.

Building the Web Tier
+++++++++++++++++++++

1. Move to your ``argovisNg`` folder which you cloned earlier:

   .. code:: bash

      ~ $ cd ~/argovis/argovisNg

2. Clone the API code here:

   .. code:: bash

      argovisNg $ git clone https://github.com/argovis/argovis_backend

   At this point, ``argovisNg`` contains all the code describing the API and web frontend. Hack away on the code, and carry on when you're ready to try out what you've coded.

3. Still in the ``argovisNg`` directory, rebuild the web tier. In this example, I call the new web tier image ``argovis/argo-express:my-build``, but you can call it anything you like:

   .. code:: bash

      argovisNg $ docker image build -t argovis/argo-express:my-build .

.. admonition:: Build too slow?

   This build can take a while. Once you have it working once, try out :ref:`fast_angular_builds` for a faster development cycle.

4. Open up ``argovisNg/docker-compose.yml`` and edit the definition of the ``argo-express`` service to use your new image:

   .. code:: yaml

      version: '3'
      services:
        argo-express:
          image: argovis/argo-express:my-build
          ...

5. Redeploy your local version of the app with ``docker-compose down`` / ``docker-compose up -d``. The web tier will now be based off of the custom image you just built.