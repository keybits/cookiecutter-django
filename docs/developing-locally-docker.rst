Getting Up and Running Locally With Docker
==========================================

.. index:: Docker

The steps below will get you up and running with a local development environment.
All of these commands assume you are in the root of your generated project.

Prerequisites
-------------

You'll need at least Docker 1.10.

If you don't already have it installed, follow the instructions for your OS:

 - On Mac OS X/Windows, you'll need `Docker Toolbox`_
 - On Linux, you'll need `docker-engine`_
.. _`Docker Toolbox`: https://github.com/docker/toolbox/releases
.. _`docker-engine`: https://docs.docker.com/engine/installation/

Create the Machine (Optional)
-----------------------------

On Linux you have native Docker, so you don't need to create a VM with
docker-machine to use it.

However, on Mac/Windows/other systems without native Docker, you'll want to
start by creating a VM with docker-machine::

    $ docker-machine create --driver virtualbox dev1

**Note:** If you want to have more than one docker development environment, then
name them accordingly. Instead of 'dev1' you might have 'dev2', 'myproject',
'djangopackages', et al.

Get the IP Address
------------------

Once your machine is up and running, run this::

    $ docker-machine ip dev1
    123.456.789.012

This is also the IP address where the Django project will be served from.

Build the Stack
---------------

This can take a while, especially the first time you run this particular command
on your development system::

    $ docker-compose -f dev.yml build

If you want to build the production environment you don't have to pass an argument -f, it will automatically use docker-compose.yml.

Boot the System
---------------

This brings up both Django and PostgreSQL.

The first time it is run it might take a while to get started, but subsequent
runs will occur quickly.

Open a terminal at the project root and run the following for local development::

    $ docker-compose -f dev.yml up

You can also set the environment variable ``COMPOSE_FILE`` pointing to ``dev.yml`` like this::

    $ export COMPOSE_FILE=dev.yml

And then run::

    $ docker-compose up

Running management commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~

As with any shell command that we wish to run in our container, this is done
using the ``docker-compose run`` command.

To migrate your app and to create a superuser, run::

    $ docker-compose -f dev.yml run django python manage.py migrate
    $ docker-compose -f dev.yml run django python manage.py createsuperuser

Here we specify the ``django`` container as the location to run our management commands.

Production Mode
~~~~~~~~~~~~~~~

Instead of using `dev.yml`, you would use `docker-compose.yml`.

Database Backups
~~~~~~~~~~~~~~~~

The database has to be running to create/restore a backup.

First, run the app with `docker-compose -f dev.yml up`.

To create a backup, run::

    docker-compose -f dev.yml run postgres backup


To list backups, run::

    docker-compose -f dev.yml run postgres list-backups


To restore a backup, run::

    docker-compose -f dev.yml run postgres restore filename.sql

To copy the files from the running Postgres container to the host system::

    docker cp <containerId>:/backups /host/path/target

Where <containerId> is the ID of the Postgres container. To get it, run::

    docker ps

Other Useful Tips
-----------------

Make a machine the active unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This tells our computer that all future commands are specifically for the dev1 machine.
Using the ``eval`` command we can switch machines as needed.

::

    $ eval "$(docker-machine env dev1)"

Detached Mode
~~~~~~~~~~~~~

If you want to run the stack in detached mode (in the background), use the ``-d`` argument:

::

    $ docker-compose -f dev.yml up -d

Debugging
~~~~~~~~~~~~~

ipdb
"""""

If you are using the following within your code to debug:

::

    import ipdb; ipdb.set_trace()

Then you may need to run the following for it to work as desired:

::

    $ docker-compose run -f dev.yml --service-ports django


django-debug-toolbar
""""""""""""""""""""

In order for django-debug-toolbar to work with docker you need to add your docker-machine ip address (the output of `Get the IP ADDRESS`_) to INTERNAL_IPS in local.py


.. May be a better place to put this, as it is not Docker specific.

You may need to add the following to your css in order for the django-debug-toolbar to be visible (this applies whether Docker is being used or not):

.. code-block:: css

    /* Override Bootstrap 4 styling on Django Debug Toolbar */
    #djDebug[hidden], #djDebug [hidden] {
        display: block !important;
    }

    #djDebug [hidden][style='display: none;'] {
        display: none !important;
    }


Using the Mailhog Docker Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In development you can (optionally) use MailHog_ for email testing. If you selected `use_docker`, MailHog is added as a Docker container. To use MailHog:

1. Make sure, that ``mailhog`` docker container is up and running
2. Open your browser and go to ``http://127.0.0.1:8025``

.. _Mailhog: https://github.com/mailhog/MailHog/
