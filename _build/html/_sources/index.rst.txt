.. argovis documentation master file, created by
   sphinx-quickstart on Wed Sep  8 04:33:12 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Argovis Documentation
=====================

Welcome to the documentation for the Argovis project!

Conventions
-----------

We follow the following conventions in these docs:

Code prompts
++++++++++++

Unless otherwise noted, all terminal commands are *bash shell commands*.

Prompts indicate the directory the command is expected to be run from, and are delimited with a ``$`` character. So:

.. code:: bash

   documentation $ pwd

indicates you're expected to run the ``pwd`` command in the ``documentation`` directory. Note that only the 'nearest' directory level is noted in this way, for brevity; the ``documentation`` directory in the above example might in fact be at ``/Users/lovelace/projects/documentation`` on your machine, but we hope this will be clear from context in each article.

Per the typical convention, we use ``~`` in this context to indicate your home directory. 

Contributing
------------

We enthusiastically welcome contributions via pull request to the documentation `on GitHub <https://github.com/argovis/documentation>`_. Follow these steps to get set up contributing documentation:

0. Before investing a lot of your time writing docs, `open an issue in the docs repo <https://github.com/argovis/documentation/issues>`_ to discuss your idea or request with the team, so we can make sure it's a good fit for the project and no one else is already working on it.

1. Make sure you have installed on your development machine:

 - git
 - docker
 - a bash shell

You'll also need a free GitHub account.

2. Fork https://github.com/argovis/documentation on GitHub to your own account.

3. Clone your fork of the documentation repo to your local machine.

4. Try building the docs as is:

.. code:: bash

   ~ $ cd documentation
   documentation $ docker container run -it -v $(pwd):/src arogvis/docbuilder:dev make html

If all is well, open ``documentation/_build/html/index.html`` in your browser to see your locally-built version of the docs.

5. From here, you're set up and ready to start creating your documentation. When you're done, commit your code, push it back to your fork on GitHub, and open a pull request.

.. toctree::
   :maxdepth: 2
   :caption: Getting Started

   getting_started/set_up_dev_environment

.. toctree::
   :maxdepth: 2
   :caption: Disaster Recovery

   disaster_recovery/restore_a_mongo_collection

.. toctree::
   :maxdepth: 2
   :caption: Special Topics

   special_topics/connecting_to_the_container_network
   special_topics/goship-demo
