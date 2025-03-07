 .. Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

 ..   http://www.apache.org/licenses/LICENSE-2.0

 .. Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.


Installation
------------

.. contents:: :local:

This page describes installations using the ``apache-airflow`` package `published in
PyPI <https://pypi.org/project/apache-airflow/>`__, but some information may be useful during
installation with other tools as well.

.. note::

    Airflow is also distributed as a Docker image (OCI Image). Consider using it to guarantee that software will always run the same no matter where it is deployed. For more information, see: :doc:`docker-stack:index`.

Version Life Cycle
''''''''''''''''''

Apache Airflow version life cycle:

+---------+-----------------+---------------+-----------------+----------------+
| Version | State           | First Release | Limited Support | EOL/Terminated |
+---------+-----------------+---------------+-----------------+----------------+
| 2       | Supported       | Dec 17, 2020  | Dec 2021        | TBD            |
+---------+-----------------+---------------+-----------------+----------------+
| 1.10    | Limited Support | Aug 27, 2018  | Dec 17, 2020    | June 2021      |
+---------+-----------------+---------------+-----------------+----------------+
| 1.9     | EOL             | Jan 03, 2018  | Aug 27, 2018    | Aug 2018       |
+---------+-----------------+---------------+-----------------+----------------+
| 1.8     | EOL             | Mar 19, 2017  | Jan 03, 2018    | Jan 2018       |
+---------+-----------------+---------------+-----------------+----------------+
| 1.7     | EOL             | Mar 28, 2016  | Mar 19, 2017    | Mar 2017       |
+---------+-----------------+---------------+-----------------+----------------+

Limited support versions will be supported with security and critical bug fix only.
EOL versions will not get any fixes nor support.
We **highly** recommend installing the latest Airflow release which has richer features.


Prerequisites
'''''''''''''

Airflow is tested with:

* Python: 3.6, 3.7, 3.8

* Databases:

  * PostgreSQL:  9.6, 10, 11, 12, 13
  * MySQL: 5.7, 8
  * SQLite: 3.15.0+
  * MSSQL(Experimental): 2017, 2019

* Kubernetes: 1.18.15 1.19.7 1.20.2

**Note:** MySQL 5.x versions are unable to or have limitations with
running multiple schedulers -- please see: :doc:`/concepts/scheduler`. MariaDB is not tested/recommended.

**Note:** SQLite is used in Airflow tests. Do not use it in production. We recommend
using the latest stable version of SQLite for local development.

Please note that with respect to Python 3 support, Airflow 2.0.0 has been
tested with Python 3.6, 3.7, and 3.8, but does not yet support Python 3.9.

Installation tools
''''''''''''''''''

Only ``pip`` installation is currently officially supported.

While there are some successes with using other tools like `poetry <https://python-poetry.org/>`_ or
`pip-tools <https://pypi.org/project/pip-tools/>`_, they do not share the same workflow as
``pip`` - especially when it comes to constraint vs. requirements management.
Installing via ``Poetry`` or ``pip-tools`` is not currently supported. If you wish to install airflow
using those tools you should use the :ref:`constraint files <installation:constraints>`  and convert them to appropriate
format and workflow that your tool requires.

.. _installation:airflow_extra_dependencies:

Airflow extra dependencies
''''''''''''''''''''''''''

The ``apache-airflow`` PyPI basic package only installs what's needed to get started.
Additional packages can be installed depending on what will be useful in your
environment. For instance, if you don't need connectivity with Postgres,
you won't have to go through the trouble of installing the ``postgres-devel``
yum package, or whatever equivalent applies on the distribution you are using.

Most of the extra dependencies are linked to a corresponding provider package. For example "amazon" extra
has a corresponding ``apache-airflow-providers-amazon`` provider package to be installed. When you install
Airflow with such extras, the necessary provider packages are installed automatically (latest versions from
PyPI for those packages). However you can freely upgrade and install provider packages independently from
the main Airflow installation.

For the list of the extras and what they enable, see: :doc:`extra-packages-ref`.

.. _installation:provider_packages:

Provider packages
'''''''''''''''''

Unlike Apache Airflow 1.10, the Airflow 2.0 is delivered in multiple, separate, but connected packages.
The core of Airflow scheduling system is delivered as ``apache-airflow`` package and there are around
60 provider packages which can be installed separately as so called ``Airflow Provider packages``.
The default Airflow installation doesn't have many integrations and you have to install them yourself.

You can even develop and install your own providers for Airflow. For more information,
see: :doc:`apache-airflow-providers:index`

For the list of the provider packages and what they enable, see: :doc:`apache-airflow-providers:packages-ref`.

Differences between extras and providers
''''''''''''''''''''''''''''''''''''''''

Just to prevent confusion of extras versus provider packages: Extras and providers are different things,
though many extras are leading to installing providers.

Extras are standard Python setuptools feature that allows to add additional set of dependencies as
optional features to "core" Apache Airflow. One of the type of such optional features are providers
packages, but not all optional features of Apache Airflow have corresponding providers.

We are using the ``extras`` setuptools features to also install provider packages.
Most of the extras are also linked (same name) with provider packages - for example adding ``[google]``
extra also adds ``apache-airflow-providers-google`` as dependency. However there are some extras that do
not install providers (examples ``github_enterprise``, ``kerberos``, ``async`` - they add some extra
dependencies which are needed for those ``extra`` features of Airflow mentioned. The three examples
above add respectively github enterprise oauth authentication, kerberos integration or
asynchronous workers for gunicorn. None of those have providers, they are just extending Apache Airflow
"core" package with new functionalities.

System dependencies
'''''''''''''''''''

You need certain system level requirements in order to install Airflow. Those are requirements that are known
to be needed for Linux system (Tested on Ubuntu Buster LTS) :

.. code-block:: bash
   :substitutions:

   sudo apt-get install -y --no-install-recommends \
           freetds-bin \
           krb5-user \
           ldap-utils \
           libffi6 \
           libsasl2-2 \
           libsasl2-modules \
           libssl1.1 \
           locales  \
           lsb-release \
           sasl2-bin \
           sqlite3 \
           unixodbc

You also need database client packages (Postgres or MySQL) if you want to use those databases.

.. _installation:constraints:

Constraints files
'''''''''''''''''

Airflow installation might be sometimes tricky because Airflow is a bit of both a library and application.
Libraries usually keep their dependencies open and applications usually pin them, but we should do neither
and both at the same time. We decided to keep our dependencies as open as possible
(in ``setup.cfg`` and ``setup.py``) so users can install different
version of libraries if needed. This means that from time to time plain ``pip install apache-airflow`` will
not work or will produce unusable Airflow installation.

In order to have repeatable installation, starting from **Airflow 1.10.10** and updated in
**Airflow 1.10.13** we also keep a set of "known-to-be-working" constraint files in the
``constraints-main``, ``constraints-2-0`` orphan branches and then we create tag
for each released version e.g. :subst-code:`constraints-|version|`. This way, when we keep a tested and working set of dependencies.

Those "known-to-be-working" constraints are per major/minor Python version. You can use them as constraint
files when installing Airflow from PyPI. Note that you have to specify correct Airflow version
and Python versions in the URL.

You can create the URL to the file substituting the variables in the template below.

.. code-block::

  https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt

where:

- ``AIRFLOW_VERSION`` - Airflow version (e.g. :subst-code:`|version|`) or ``main``, ``2-0``, for latest development version
- ``PYTHON_VERSION`` Python version e.g. ``3.8``, ``3.7``

There is also a no-providers constraint file, which contains just constraints required to install Airflow core. This allows
to install and upgrade airflow separately and independently from providers.

You can create the URL to the file substituting the variables in the template below.

.. code-block::

  https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-no-providers-${PYTHON_VERSION}.txt

Installation script
'''''''''''''''''''

In order to simplify the installation, we have prepared examples that will select
:ref:`the constraints file <installation:constraints>` compatible with your Python version.

**Installing Airflow with extras and providers**

If you need to install :ref:`extra dependencies of airflow <installation:airflow_extra_dependencies>`,
you can use the script below to make an installation a one-liner (the example below installs
postgres and google provider, as well as ``async`` extra.

.. code-block:: bash
    :substitutions:

    AIRFLOW_VERSION=|version|
    PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
    pip install "apache-airflow[async,postgres,google]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

Note, that it will install the versions of providers that were available at the moment this version of Airflow
has been prepared. You need to follow next steps if you want to upgrade provider packages in case they were
released afterwards.


**Upgrading Airflow with providers**

You can also upgrade airflow together with extras (providers available at the time of the release of Airflow
being installed.


.. code-block:: bash
    :substitutions:

    AIRFLOW_VERSION=|version|
    PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
    pip install --upgrade "apache-airflow[postgres,google]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

**Installation and upgrading of Airflow providers separately**

You can manually install all the providers you need. You can continue using the "providers" constraint files
but the 'versioned' airflow constraints installs only the versions of providers that were available in PyPI at
the time of preparing of the airflow version. However, usually you can use "main" version of the providers
to install latest version of providers. Usually the providers work with most versions of Airflow, if there
will be any incompatibilities, it will be captured as package dependencies.

.. code-block:: bash

    PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
    # For example: 3.6
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-main/constraints-${PYTHON_VERSION}.txt"
    pip install "apache-airflow-providers-google" --constraint "${CONSTRAINT_URL}"

You can also upgrade the providers to latest versions (you need to use main version of constraints for that):

.. code-block:: bash

    PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
    # For example: 3.6
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-main/constraints-${PYTHON_VERSION}.txt"
    pip install "apache-airflow-providers-google" --upgrade --constraint "${CONSTRAINT_URL}"


**Installation and upgrade of Airflow core:**

If you don't want to install any extra providers, initially you can use the command set below.

.. code-block:: bash
    :substitutions:

    AIRFLOW_VERSION=|version|
    PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
    # For example: 3.6
    CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-no-providers-${PYTHON_VERSION}.txt"
    # For example: https://raw.githubusercontent.com/apache/airflow/constraints-|version|/constraints-no-providers-3.6.txt
    pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"


Support for Python and Kubernetes versions
''''''''''''''''''''''''''''''''''''''''''

As of Airflow 2.0 we agreed to certain rules we follow for Python and Kubernetes support.
They are based on the official release schedule of Python and Kubernetes, nicely summarized in the
`Python Developer's Guide <https://devguide.python.org/#status-of-python-branches>`_ and
`Kubernetes version skew policy <https://kubernetes.io/docs/setup/release/version-skew-policy>`_.

1. We drop support for Python and Kubernetes versions when they reach EOL. We drop support for those
   EOL versions in main right after EOL date, and it is effectively removed when we release the
   first new MINOR (Or MAJOR if there is no new MINOR version) of Airflow
   For example for Python 3.6 it means that we drop support in main right after 23.12.2021, and the first
   MAJOR or MINOR version of Airflow released after will not have it.

2. The "oldest" supported version of Python/Kubernetes is the default one. "Default" is only meaningful
   in terms of "smoke tests" in CI PRs which are run using this default version and default reference
   image available in DockerHub. Currently ``apache/airflow:latest`` and ``apache/airflow:2.0.2`` images
   are both Python 3.6 images, however the first MINOR/MAJOR release of Airflow release after 23.12.2021 will
   become Python 3.7 images.

3. We support a new version of Python/Kubernetes in main after they are officially released, as soon as we
   make them work in our CI pipeline (which might not be immediate due to dependencies catching up with
   new versions of Python mostly) we release a new images/support in Airflow based on the working CI setup.

Installing Airflow From Released Sources and packages
'''''''''''''''''''''''''''''''''''''''''''''''''''''

You can also install Airflow using the official sources and packages. Those sources and packages
released are the "official" sources of installation that you can use if you want to verify the
origin of the packages and want to verify checksums and signatures of the packages.

The packages are available at the
`Official Apache Software Foundations Downloads page <https://downloads.apache.org/airflow/>`_

The |version| downloads are available at:

* `Apache Airflow |version| sdist package <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-bin.tar.gz>`_ (`asc <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-bin.tar.gz.asc>`__, `sha512 <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-bin.tar.gz.sha512>`__)
* `Apache Airflow |version| wheel package <https://downloads.apache.org/airflow/|version|/apache_airflow-|version|-py3-none-any.whl>`_ (`asc <https://downloads.apache.org/airflow/|version|/apache_airflow-|version|-py3-none-any.whl.asc>`__, `sha512 <https://downloads.apache.org/airflow/|version|/apache_airflow-|version|-py3-none-any.whl.sha512>`__)
* `Apache Airflow |version| sources <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-source.tar.gz>`_ (`asc <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-source.tar.gz.asc>`__, `sha512 <https://downloads.apache.org/airflow/|version|/apache-airflow-|version|-source.tar.gz.sha512>`__)


Set up a database
'''''''''''''''''

Airflow requires a database. If you're just experimenting and learning Airflow, you can stick with the
default SQLite option. If you don't want to use SQLite, then take a look at
:doc:`howto/set-up-database` to setup a different database.


Troubleshooting
'''''''''''''''

This section describes how to troubleshoot installation issues.

Airflow command is not recognized
"""""""""""""""""""""""""""""""""

If the ``airflow`` command is not getting recognized (can happen on Windows when using WSL), then
ensure that ``~/.local/bin`` is in your ``PATH`` environment variable, and add it in if necessary:

.. code-block:: bash

    PATH=$PATH:~/.local/bin

You can also start airflow with ``python -m airflow``

``Symbol not found: _Py_GetArgcArgv``
"""""""""""""""""""""""""""""""""""""

If you see ``Symbol not found: _Py_GetArgcArgv`` while starting or importing Airflow, this may mean that you are using an incompatible version of Python.
For a homebrew installed version of Python, this is generally caused by using Python in ``/usr/local/opt/bin`` rather than the Frameworks installation (e.g. for ``python 3.7``: ``/usr/local/opt/python@3.7/Frameworks/Python.framework/Versions/3.7``).

The crux of the issue is that a library Airflow depends on, ``setproctitle``, uses a non-public Python API
which is not available from the standard installation ``/usr/local/opt/`` (which symlinks to a path under ``/usr/local/Cellar``).

An easy fix is just to ensure you use a version of Python that has a dylib of the Python library available. For example:

.. code-block:: bash

  # Note: these instructions are for python3.7 but can be loosely modified for other versions
  brew install python@3.7
  virtualenv -p /usr/local/opt/python@3.7/Frameworks/Python.framework/Versions/3.7/bin/python3 .toy-venv
  source .toy-venv/bin/activate
  pip install apache-airflow
  python
  >>> import setproctitle
  # Success!

Alternatively, you can download and install Python directly from the `Python website <https://www.python.org/>`__.
