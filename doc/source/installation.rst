Installation
============

To install Jenkins Job Builder from source, run::

  pip install --user jenkins-job-builder

A virtual environment is recommended for development.  For example, Jenkins
Job Builder may be installed from the top level directory::

    $ virtualenv .venv
    $ source .venv/bin/activate
    $ pip install -r test-requirements.txt -e .

Alternatively, the current release can be installed from pypi::

  sudo pip install jenkins-job-builder

The OpenStack project uses Puppet to manage its infrastructure
systems, including Jenkins.  If you use Puppet, you can use the
`OpenStack Jenkins module`__ to install Jenkins Job Builder.

__ https://opendev.org/opendev/puppet-jenkins/src/branch/master

Documentation
-------------

Documentation is included in the ``doc`` folder. To generate docs
locally execute the command::

    tox -e docs

The generated documentation is then available under
``doc/build/html/index.html``.

As over time URLs change or become stale there is also a testenv available
to verify any links added. To run locally execute the command::

    tox -e docs-linkcheck

* Note: When behind a proxy it is necessary to use ``TOX_TESTENV_PASSENV``
  to pass any proxy settings for this test to be able to check links are
  valid.

Unit Tests
----------

Unit tests have been included and are in the ``tests`` folder.  We recently
started including unit tests as examples in our documentation so to keep the
examples up to date it is very important that we include unit tests for
every module.  To run the unit tests, execute the command::

    tox -e py27

* Note: View ``tox.ini`` to run tests on other versions of Python.

Test Coverage
-------------

To measure test coverage, execute the command::

    tox -e cover
