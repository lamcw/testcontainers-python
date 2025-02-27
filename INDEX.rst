testcontainers-python
=====================

.. image:: https://github.com/testcontainers/testcontainers-python/workflows/testcontainers-python/badge.svg
   :target: https://github.com/testcontainers/testcontainers-python/actions/workflows/main.yml
.. image:: https://img.shields.io/pypi/v/testcontainers.svg
   :target: https://pypi.python.org/pypi/testcontainers
.. image:: https://readthedocs.org/projects/testcontainers-python/badge/?version=latest
   :target: http://testcontainers-python.readthedocs.io/en/latest/?badge=latest
.. image:: https://github.com/codespaces/badge.svg
   :target: https://codespaces.new/testcontainers/testcontainers-python

testcontainers-python facilitates the use of Docker containers for functional and integration testing. The collection of packages currently supports the following features.

.. toctree::

    core/README
    modules/arangodb/README
    modules/azurite/README
    modules/clickhouse/README
    modules/elasticsearch/README
    modules/google/README
    modules/kafka/README
    modules/keycloak/README
    modules/localstack/README
    modules/minio/README
    modules/mongodb/README
    modules/mssql/README
    modules/mysql/README
    modules/neo4j/README
    modules/nginx/README
    modules/opensearch/README
    modules/oracle/README
    modules/postgres/README
    modules/rabbitmq/README
    modules/redis/README
    modules/selenium/README
    modules/k3s/README

Getting Started
---------------

.. doctest::

    >>> from testcontainers.postgres import PostgresContainer
    >>> import sqlalchemy

    >>> with PostgresContainer("postgres:latest") as postgres:
    ...     psql_url = postgres.get_connection_url()
    ...     engine = sqlalchemy.create_engine(psql_url)
    ...     with engine.begin() as connection:
    ...         result = connection.execute(sqlalchemy.text("select version()"))
    ...         version, = result.fetchone()
    >>> version
    'PostgreSQL ...'

The snippet above will spin up the current latest version of a postgres database in a container. The :code:`get_connection_url()` convenience method returns a :code:`sqlalchemy` compatible url (using the :code:`psycopg2` driver per default) to connect to the database and retrieve the database version.

.. doctest::

    >>> import asyncpg
    >>> from testcontainers.postgres import PostgresContainer

    >>> with PostgresContainer("postgres:16", driver=None) as postgres:
    ...     psql_url = container.get_connection_url()
    ...     with asyncpg.create_pool(dsn=psql_url,server_settings={"jit": "off"}) as pool:
    ...         conn = await pool.acquire()
    ...         ret = await conn.fetchval("SELECT 1")
    ...         assert ret == 1

This snippet does the same, however using a specific version and the driver is set to None, to influence the :code:`get_connection_url()` convenience method to not include a driver in the URL (e.g. for compatibility with :code:`psycopg` v3).

Note, that the :code:`sqlalchemy` and :code:`psycopg2` packages are no longer a dependency of :code:`testcontainers[postgres]` and not needed to launch the Postgres container. Your project therefore needs to declare a dependency on the used driver and db access methods you use in your code.


Installation
------------

The suite of testcontainers packages is available on `PyPI <https://pypi.org/project/testcontainers/>`_,
and individual packages can be installed using :code:`pip`.

Version `4.0.0` onwards we do not support the `testcontainers-*` packages as it is unsutainable to maintain ownership.

Instead packages can be installed by specifying `extras <https://setuptools.readthedocs.io/en/latest/setuptools.html#declaring-extras-optional-features-with-their-own-dependencies>`__, e.g., :code:`pip install testcontainers[postgres]`.


Docker in Docker (DinD)
-----------------------

When trying to launch a testcontainer from within a Docker container, e.g., in continuous integration testing, two things have to be provided:

1. The container has to provide a docker client installation. Either use an image that has docker pre-installed (e.g. the `official docker images <https://hub.docker.com/_/docker>`_) or install the client from within the `Dockerfile` specification.
2. The container has to have access to the docker daemon which can be achieved by mounting `/var/run/docker.sock` or setting the `DOCKER_HOST` environment variable as part of your `docker run` command.

Development and Contributing
----------------------------

We recommend you use a `virtual environment <https://virtualenv.pypa.io/en/stable/>`_ for development (:code:`python>=3.7` is required). After setting up your virtual environment, you can install all dependencies and test the installation by running the following snippet.

.. code-block:: bash

    poetry install --all-extras
    make <your-module>/tests

Package Structure
^^^^^^^^^^^^^^^^^

Testcontainers is a collection of `implicit namespace packages <https://peps.python.org/pep-0420/>`__ to decouple the development of different extensions, e.g., :code:`testcontainers-mysql` and :code:`testcontainers-postgres` for MySQL and PostgreSQL database containers, respectively. The folder structure is as follows.

.. code-block:: bash

      modules
      # One folder per feature.
      [feature name]
          # Folder without __init__.py for implicit namespace packages.
          testcontainers
              # Implementation as namespace package with __init__.py.
              [feature name]
                  __init__.py
                  # Other files for this
                  ...
          # Tests for the feature.
          tests
              test_[some_aspect_for_the_feature].py
              ...
          # README for this feature.
          README.rst
          # Setup script for this feature.
          setup.py

Contributing a New Feature
^^^^^^^^^^^^^^^^^^^^^^^^^^

You want to contribute a new feature or container? Great! You can do that in six steps as outlined `here <https://github.com/testcontainers/testcontainers-python/blob/main/.github/PULL_REQUEST_TEMPLATE/new_container.md>__`.
