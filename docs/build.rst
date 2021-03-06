==============
Anaconda Build
==============

Overview
========

How Anaconda Build works
~~~~~~~~~~~~~~~~~~~~~~~~

The **Anaconda Build System** provides **Continuous Integration**
powered by **conda** and **conda environments** for all platforms,
through a simple yet powerful interface.

The build process follows an **Anaconda Build Recipe** (a set of
instructions specified in a ``.binstar.yml`` file), which includes the
(conda) test environments to use, build instructions and additional
scripts to be executed. A **Build Recipe** does not require the creation
of a **conda package** as a final product. However, creating **conda
packages** and uploading them to Anaconda Cloud provides a seamless and
comprehensive workflow for the maintenance and distribution of packages.


Why use Anaconda Build?
~~~~~~~~~~~~~~~~~~~~~~~

There are several use cases for which Anaconda Build can be used to
provide an efficient packaging workflow. A user may:

-  Have a series of packages and want to compile them into conda
   packages so it's easy for other users to conda install those packages
   from the first user's channel on Anaconda Cloud.

-  Be part of a workgroup or team using a GitHub repository that has
   basic Continuous Integration needs that can be centralized with
   Anaconda Cloud so other potential users can easily download their
   open source packages for different platforms.

-  Be part of an organization that needs to build and distribute
   cross-platform conda packages across OS X, Windows and Linux.


System components
~~~~~~~~~~~~~~~~~

The **Anaconda Cloud Build System** is based on three main components:


Build queues
^^^^^^^^^^^^

The **Build Queues** receive build job requests from users and assign
these requests to the different available workers. Queues are hosted on
**Anaconda Cloud**.

Each Anaconda Cloud user may use the public Linux-64 build queue
provided by Anaconda Cloud. New build queues can be created by
organizations on Anaconda Cloud.

When an organization creates a queue no groups have access to it, and
the organization can grant access to that queue to one or more groups.
The organization can create groups, add organizations and individual
users to a group or remove them, and can turn on or off a group's access
to a queue.


Workers
^^^^^^^

These are machines that perform the actual build instructions and
testing specified in the **Anaconda Build Recipe**. They are often
Amazon Web Services (AWS) instances. Different workers can be created
for different platforms including:

-  Windows 32/64
-  Linux 32/64
-  OS X 64

Note: By default only Linux-64 build workers are available for public
use with Anaconda Cloud. For access to additional platforms you can add
your own build workers on your local machine, in virtual machines (VMs)
or on cloud computing providers such as AWS.


Users
^^^^^

Users are responsible for creating and launching anaconda builds from
the command line interface into a (previously created) build queue.

To enable a user to use a queue other than the Anaconda Cloud public
Linux-64 queue, the organization that created the queue must add the
user to a group and grant that group access to the queue.


Builds in Anaconda Cloud
========================

Prerequisites
~~~~~~~~~~~~~

Before using the Anaconda Build System:

#. Create an account on Anaconda Cloud at
   `anaconda.org <https://anaconda.org/>`__
#. Install `Anaconda <https://www.continuum.io/downloads>`__ or
   `Miniconda <http://conda.pydata.org/miniconda.html>`__
#. Install conda build, Anaconda Client and Anaconda Build:
   ``conda install conda-build anaconda-client anaconda-build``
#. Log in to Anaconda Cloud from the command line using Anaconda Client:
   ``anaconda login``
#. (**Optional**) Create an organization to be able to create new
   queues, at
   `anaconda.org/new/organization <https://anaconda.org/new/organization>`__

.. _submit-a-build:

Submit a build
~~~~~~~~~~~~~~

Create a package
^^^^^^^^^^^^^^^^

If you are not familiar with anaconda-client, create a package first.
This will be the namespace or the context of the build.

::

    mkdir conda_build_test
    cd conda_build_test
    anaconda package --create USERNAME/conda_build_test


Create a build config
^^^^^^^^^^^^^^^^^^^^^

Let's create a simple build config file and submit an empty build job to
illustrate the basic steps.

The ``.binstar.yml`` file holds the Anaconda Build Recipe for how to
build and test your package. It tells the build system which platforms
to build on, which environment variables to use and which scripts to
execute.

To add a ``.binstar.yml`` file to your working directory, run
``anaconda build init`` in your working directory.

Note: This should be the same directory as your ``meta.yaml`` file if
you are building a conda package.

Once this is complete you should be able to submit your first build that
will print ``This is my anaconda build!``

::

    anaconda build init
    anaconda build submit .
    anaconda build tail -f USERNAME/conda_build_test 1

We have just created an empty package with a single Build Recipe
instruction, namely printing ``This is my anaconda build!``.


Create conda package
^^^^^^^^^^^^^^^^^^^^

Let's create a conda package to show that anaconda build can do some
actual work.

You need to add a ``meta.yaml`` file and modify your ``.binstar.yml``
file so it contains the following keys:

.binstar.yml::

  package: conda_build_test
  script:
    - conda build .
  build_targets: conda

meta.yaml::

  package:
    name: conda_build_test
    version: 0.0.1
  build:
    number: 1
    script:
      - echo "This is my anaconda build with conda"
  requirements:
    run:
      - python
  about:
    summary: This is an anaconda build test!

Note: Please see our publicly available `Conda
Recipes <https://github.com/conda/conda-recipes>`__.


Submit your conda build
^^^^^^^^^^^^^^^^^^^^^^^

Once your have created the ``meta.yaml`` file you can test that your
conda build runs locally with `conda
build <http://conda.pydata.org/docs/build.html>`__.

Submitting this build is the same as the first::

    conda build .
    anaconda build submit .
    anaconda build tail -f USERNAME/conda_build_test 2


Install your new conda package
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default Anaconda Cloud puts all new packages in a ``dev`` label in
your account. See :ref:`using-labels-in-the-development-cycle` for a more in
depth example on how to use labels.

::

    conda install -c USERNAME/label/dev conda_build_test

.. _github-builds:

GitHub builds
~~~~~~~~~~~~~

Create a git repo
^^^^^^^^^^^^^^^^^

Let's use the package you have created in the :ref:`submit a
build <submit-a-build>` example. First `create a new github
repository <https://github.com/new>`__. You will then need to push the
files to github.

First: `create a new github repository <https://github.com/new>`__

::

    git init
    git add * .binstar.yml
    git commit -m "first commit"
    git remote add origin https://github.com/GITHUB_USERNAME/conda_build_test.git
    git push -u origin master


Submit the build
^^^^^^^^^^^^^^^^

Once the package source is pushed to github you can submit a build via a
github url.

::

    anaconda build submit https://github.com/GITHUB_USERNAME/conda_build_test

.. _save-and-trigger-builds:

Save and trigger your builds
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you have :ref:`submitted a build from github <github-builds>` you may
want to save your build configuration, especially if you are using
:ref:`extra options <cli-submit>` like ``-p``, ``--sub-dir``,
``--label``, ``--queue`` or ``--email``.

You can :ref:`cli-save` these options to Anaconda Cloud and
:ref:`cli-trigger` them later.

Note: Using the anaconda build save command affects the Continuous
Integration (CI) section of the package settings on Anaconda Cloud. For
example, running an "anaconda build save" command that uses the
"--label" flag will update the label used by CI services. The CI section
of the package settings can be seen by going to the package's page on
Anaconda Cloud and choosing "Settings" and then "Continuous
Integration", or by examining the package's binstar.yaml file.

::

    anaconda build save -p USERNAME/conda_build_test https://github.com/GITHUB_USERNAME/conda_build_test --label dev
    anaconda build trigger USERNAME/conda_build_test

It is also possible to trigger a build on a specific queue, build host,
repository branch, and/or distribution. For information on options, see

::

    anaconda build trigger --help

Here is an example of triggering a build on a queue, repository branch,
and centos distribution:

::

    anaconda build trigger USERNAME/conda_build_test --dist centos --branch master --queue USERNAME/QUEUENAME


Continuous Integration: run a build on git push
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you have saved a build you can view the information on the website
at

``https://anaconda.org/USERNAME/conda_build_test/settings/ci``

To get to this page, navigate to your package
(``https://anaconda.org/USERNAME/conda_build_test``). Then choose
``settings`` and ``Continuous Integration``.

|Continuous Integration page|

Click "Edit". The fields we care about for enabling continuous
integration are:

branches to test: This is a python regular expression (regex) describing what branches
should trigger builds in ``test-only`` mode. No files will be uploaded
to Anaconda Cloud.

branches to upload: This is a python regex describing what branches should trigger builds
that also upload the resulting :ref:`build_targets <build-targets>`.

** This can cause many files to accumulate in your account. Use
carefully.

Add Webhook: If checked Anaconda Cloud will add a `github
webhook <https://developer.github.com/webhooks/>`__ with the value
`https://api.anaconda.org/github-hook <https://api.anaconda.org/github-hook>`__
to your github package.

For this example:

-  Set ``Test Branches`` to ``refs/heads/.*``, which matches all git
   branches.
-  Leave ``Upload Branches`` empty.
-  Make sure ``Add Webhook`` is checked.

You should see an active webhook at the end of this process.

|Webhook Continuous Integration page|

Now, test that the web hook is correct by pushing an empty commit.

::

  git commit -m "Trigger build" --allow-empty
  git push
  
  # This should give enough time to let github send the webhook
  sleep 10
  
  anaconda build list-all USERNAME/conda_build_test

To debug webhooks, first submit your build again with :ref:`anaconda-client
trigger <save-and-trigger-builds>`. This should highlight the issue
with your build.

If ``anaconda trigger`` works, but the webhook still does not, go to
github and inspect the webhook requests and responses.


Build configuration
===================

Configuration file tags
~~~~~~~~~~~~~~~~~~~~~~~

Each package you build will have a build config file in its root
directory named ``.binstar.yml``. If you are not already familiar with
the build process, please begin by reading this guide on :ref:`how to submit
a build <submit-a-build>`.

This yaml file contains a number of tags to control the way a build is
run. Every tag is optional, and all tags can be written as a single
command or as a list.

::

    tag: single_command
    # OR
    tag:
      - some_command
      - another_command

.. _script:

script
^^^^^^

Define the main script to run on the build machine:

::

    script: echo "hello world!"

Script may also be a list:

::

    script:
      - some_command
      - another_command

before\_script and after\_script
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also define scripts to be run before and after the main script:

::

    before_script: some_command
    after_script:  another_command

For the ``after_script`` tag the environment variable
:ref:`BINSTAR_BUILD_RESULT <environment-variables>` will be made
available as either *success* or *failure*.


after\_success and after\_failure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you use the after\_success or after\_failure tags, one or the other
of them will run after the :ref:`script` tags depending on if the
build was a success or a failure. **Build errors are not caught**.

::

    after_success:
      - echo Yay!
    after_failure:
      - echo Oops?

.. _build-targets:

build_targets
^^^^^^^^^^^^^

These files will be uploaded to your Anaconda Cloud package. These are
files that will be uploaded to Anaconda Cloud with :ref:`anaconda
upload <cli-upload>`.

You may use the key words ``conda`` or ``pypi``:

::

      build_targets: conda

Or a file or glob of files:

::

    build_targets: /opt/anaconda/my-package.tar.bz2

.. _build-platform:

platform
^^^^^^^^

This selects the platforms for which you wish to build your packages.

**Please note:** by default only ``linux-64`` build-workers are
available for public use on Anaconda Cloud. You can :ref:`add your own build
workers <launching-a-build-worker>` if you need access to additional
platforms.

To see which platforms are available to you, issue the :ref:`anaconda build
queue <cli-queue>` command:

::

    $ anaconda build queue
    Using anaconda-server api site https://api.anaconda.org
    
    build/binstar/public           []
     + Worker hostname:docker-2        platform:linux-64        dist:centos
       - Id 54b57d3ee1dad10a4987f6cd
       - Last seen 5 seconds ago
       - binstar-build v0.10.3 (binstar v0.10.1)
     + Worker hostname:docker-2        platform:linux-64        dist:centos
       - Id 54b989d1e1dad10da34074d6
       - Last seen 10 seconds ago
       - binstar-build v0.10.3 (binstar v0.10.1)

The anaconda-build cli has the capacity to support the following
platforms:

::

    platform:
      - linux-32
      - linux-64
      - osx-32
      - osx-64
      - win-32
      - win-64

The items in the ``platform`` tag describe the first of the three axes
of the :ref:`build-matrix`.

.. _build-engine:

engine
^^^^^^

Sets the initial conda packages you want to build with::

    engine:
      - python=2 nodejs=0.10
      - python=3

Note that the first item ``python=2 nodejs=0.10`` is not a list. In this
build item both packages python and nodejs will be available.

The items in the ``engine`` tag describe the second of the three axes of
the :ref:`build-matrix`.

The environment variables CONDA\_PY and CONDA\_NPY are set based on the
presence of Python or numpy in the engine tag.

.. _build-env:

env
^^^

An export of environment variables for the sub-build:

::

    env:
      - FOO=BAR
      - ANACONDA=GREAT JENKINS=OK

The items in the ``env`` tag describe the third of the three axes of the
:ref:`build-matrix`.


install\_channels
~~~~~~~~~~~~~~~~~

If any channels need to be added to conda for the installation, they can
be included in install\_channels.

This shows the install\_channels configured for building R packages.

::

    install_channels:
       - r
       - defaults

If private packages are required in your build, make sure to include a
:ref:`token <reference-token>` in the channel configuration.

This shows the install\_channels configured for building a package that
depends on jsmith's private packages.

::

    install_channels:
      - t/TOKEN/jsmith
      - defaults

quiet
~~~~~

The ``quiet`` key in ``.binstar.yml`` can reduce the amount of
superfluous printing to the build logs. For example, if you see
installation messages similar to the following ones in your build log,
you can redact these messages by using the ``quiet`` key.

::

    Fetching packages ...
        ncurses-5.9-1.   0% |                              | ETA:  --:--:--   0.00  B/s
        ncurses-5.9-1.   2% |                               | ETA:  0:00:00  36.09 MB/s
        ncurses-5.9-1.   4% |#                              | ETA:  0:00:00  54.15 MB/s
        ncurses-5.9-1.   6% |##                             | ETA:  0:00:00  66.78 MB/s
        ncurses-5.9-1.   9% |##                             | ETA:  0:00:00  76.02 MB/s

More specifically, the following usage of ``quiet`` in the
``.binstar.yml`` file will redact from the build log any message that
ends with ``\r``::

    quiet: True

.. _build-matrix:

Build matrix
~~~~~~~~~~~~

When you submit one ``.binstar.yml`` file many sub-builds are launched,
one for each combination of the values of the :ref:`build-platform`,
:ref:`build-engine` and :ref:`build-env` tags.

The build matrix is formed by combining ``[platform * engine * env]`` to
get the sub-builds.

The following configuration will run 8 sub builds:

::

    platform:
      - linux-32
      - linux-64
    engine:
      - python=2
      - python=3
    env:
      - CXX=g++
      - CXX=clang++

#. platform: ``linux-64`` engine: ``python=2`` env: ``CXX=g++``
#. platform: ``linux-64`` engine: ``python=2`` env: ``CXX=clang++``
#. platform: ``linux-64`` engine: ``python=3`` env: ``CXX=g++``
#. platform: ``linux-64`` engine: ``python=3`` env: ``CXX=clang++``
#. platform: ``linux-32`` engine: ``python=2`` env: ``CXX=g++``
#. platform: ``linux-32`` engine: ``python=2`` env: ``CXX=clang++``
#. platform: ``linux-32`` engine: ``python=3`` env: ``CXX=g++``
#. platform: ``linux-32`` engine: ``python=3`` env: ``CXX=clang++``


Multiple build matrices
^^^^^^^^^^^^^^^^^^^^^^^

Sometimes it is not best to define one large build matrix. For example,
if you are running a build on Windows, the matrix:

::

    platform:
      - win-32
      - linux-32
    env:
      - MSVC=2008
      - MSVC=2010
      - CC=gcc
    script:
        build.sh

would not work, because the configurations
``platform: linux-32 env: MSVC=2008`` and
``platform: linux-32 env: MSVC=2010`` don't make sense. Instead, you can
concatenate sub-builds using `yaml document
separators <http://yaml.org/spec/1.0/#id2489959>`__.

Yaml documents are separated by ``---``.

::

    platform: linux-32
    env: CC=gcc
    script: build.sh
    --- # New Build Matrix
    platform: win-32
    env:
      - MSVC=2008
      - MSVC=2010
    script: build.bat

This would now produce the correct sub-builds.


Excluding an item in the matrix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can exclude a sub-build entry from a matrix with the exclude tag.

::

    platform:
      - linux-32
      - linux-64
    engine:
      - python=2
      - python=3
    script: conda build .
    ---
    platform: linux-32
    engine: python=3
    exclude: true

Now the sub-build: ``platform: linux-32 engine: python=3`` will not be
submitted.

.. _environment-variables:

Environment variables
~~~~~~~~~~~~~~~~~~~~~

BINSTAR\_BUILD
    The build number as MAJOR.MINOR
BINSTAR\_BUILD\_MAJOR
    The major build number
BINSTAR\_BUILD\_MINOR
    The minor build number
BINSTAR\_ENGINE
    the engine from the engine tag
BINSTAR\_PLATFORM
    the platform from the platform tag
BINSTAR\_BUILD\_RESULT
    This is set after the :ref:`script` tag is run
CONDA\_PY
    The conda python version from the engine tag
CONDA\_NPY
    The conda numpy version from the engine tag


.. _build-workers:

Build workers
=============

This section contains advanced information on configuring builds on the
`Anaconda Cloud <http://anaconda.org>`__ platform. If you are not
already familiar with the build process, begin by reading this guide on
:ref:`how to submit a build <submit-a-build>`.

NOTE: Anaconda build defaults to the linux-64 platform, and `Anaconda
Cloud <http://anaconda.org>`__ provides free linux-64 workers. If you do
**not** need to build for other platforms, you can complete package
builds using only `Anaconda Cloud <http://anaconda.org>`__. Further, all
Anaconda Cloud accounts allow you to attach one free worker. If you need
more workers or workers on other platforms, you will need to :ref:`create an
organization <creating-orgs>` and `upgrade to a
paid plan <https://anaconda.org/about/pricing>`__.

Anaconda build workers allow you to run your builds on your own
machines. A build worker can run on any machine that supports bash
(posix) or batch (win32).

To follow along with this tutorial, you will need to :ref:`install the build
cli <installing-anaconda-client-and-anaconda-build>`.


Create a build queue
~~~~~~~~~~~~~~~~~~~~

The first thing you will need to do is create a build queue. A build
queue holds submitted builds until a build worker is ready to remove the
build and run it. At present, when you submit a job the default build
queue is ``binstar/public``.

To create your queue run::

    anaconda build queue --create USERNAME/QUEUENAME

Where ``USERNAME`` is your `Anaconda Cloud <http://anaconda.org>`__
username and ``QUEUENAME`` is an alphanumeric name of your choice. For
more information see :ref:`configuring your build
queues <configuring-build-queues>`.

.. _launching-a-build-worker:

Launching a build worker
~~~~~~~~~~~~~~~~~~~~~~~~

Before you can begin using the queue you created, you will need to
attach a build-worker to the queue. The basic build worker runs on your
machine (Linux, OS X or Windows) as the current user (see :ref:`security
considerations <security-considerations>`) and accepts jobs from the
build queue that you specify.

In order to avoid the build-worker waiting on user input for conda
commands, conda must be configured not to prompt for confirmation. Run
the following command to set the configuration correctly::

    conda config --set always_yes true

A worker needs to be registered before it can be run. This command
registers a worker for the queue USERNAME/QUEUE and outputs the worker's
id and other arguments to a yaml file in ~/.workers/ with the worker's
id as the yaml filename.

::

    anaconda worker register USERNAME/QUEUE

To see other options for registering workers, try

::

    anaconda worker register --help

The register step should print out a worker id you can use to run a
worker. This command will start a worker with a ``worker_id``:

::

    anaconda worker run <worker-id-from-register-step>

That's it! You can now submit a job to your queue and your new build
worker will pick it up and build it:

::

    anaconda build submit ./my-build --queue USERNAME/QUEUENAME

NOTE: You must leave your build worker process running in order to
submit builds to it. You may wish to attach build workers to your queue
using a nohup command or similar.

Finally, after killing the anaconda build worker process, it is required
to deregister the worker, unless you plan to start the worker again with
the same worker id and configuration. The deregister step can be done
with:

::

    anaconda worker deregister <worker-id-from-register-step>

If you need to deregister a worker, then check your Anaconda server
instance's /settings/build-queue page to remove the worker or list the
workers you have registered with this command:

::

    anaconda worker list

There is an option for listing only the workers registered from the
current hostname:

::

    anaconda worker list --this-host-only

Registered workers can also be filtered by queue or organization with
one of these commands:

::

    anaconda worker list --queue USERNAME/QUEUE

Or:

::

    anaconda worker list --org ORGNAME

Review all help for register, run, deregister, and list with::

    anaconda worker -h


Running workers in the background
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `Chalmers process control
system <https://github.com/Anaconda-Server/chalmers>`__ can be used to
run build workers in the background across all platforms. Please see the
readme file in that repository for further information.

.. _configuring-build-queues:

Configuring build queues
~~~~~~~~~~~~~~~~~~~~~~~~

By default your build worker will run builds on your ``binstar/public``
queue. You may change this in two ways:

#. Use the ``--queue`` option when issuing a ``anaconda build`` :ref:`submit,
   save or trigger <submitting-builds>` command::

       anaconda build submit ./my-build --queue USERNAME/QUEUENAME

#. Specify a default queue for your build workers. This will affect all
   builds for your account. You can do this by visiting
   `anaconda.org/settings/build-queue <https://anaconda.org/settings/build-queue>`__
   and clicking the **Set as Default** option for the queue you would
   like to use.


Share your build queue
^^^^^^^^^^^^^^^^^^^^^^

Once a build queue is created, you can control who may submit jobs to
it. For build queues associated with an organization rather than an
individual user, the default behavior is that only organization owners
may submit jobs to the queue.

To share access to your queue:

#. Navigate your browser to
   `anaconda.org/settings/build-queue <https://anaconda.org/settings/build-queue>`__.
   If you are an owner in multiple organizations, be sure to select the
   correct one in the drop-down in the upper right corner of the page.
#. Click on the ** icon of the queue you want to share.
#. Add the user by name (individual accounts) or by group (organization
   accounts).

.. _security-considerations:

Security considerations
~~~~~~~~~~~~~~~~~~~~~~~

Because build workers run on your machine, using the current user
account, there are a few security considerations associated with
launching a build worker. Remember the build worker runs user-defined
build scripts from the jobs that are submitted to it.

We recommend you:

#. Consider who you are giving access to your build queue.
#. Remember that the ``anaconda build worker`` builds are permanent, and
   make sure users cannot accidentally change the state of your build
   machine.


Executing builds in a Docker container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The anaconda build cli includes the ability to build in a docker
container::

    docker pull binstar/linux-64
    anaconda worker register USERNAME/QUEUENAME
    # prints worker-id
    anaconda worker docker_run <worker-id> --image binstar/linux-64

.. |Continuous Integration page| image:: /img/cloud-ci.png
.. |Webhook Continuous Integration page| image:: /img/cloud-webhook-ci.png
