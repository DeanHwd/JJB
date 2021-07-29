Configuration File
------------------

After installation, you will need to create a configuration file.  By
default, ``jenkins-jobs`` looks for ``~/.config/jenkins_jobs/jenkins_jobs.ini``,
``<script directory>/jenkins_jobs.ini`` or ``/etc/jenkins_jobs/jenkins_jobs.ini``
(in that order), but you may specify an alternative location when running
``jenkins-jobs``.  The file should have the following format:

.. literalinclude:: ../../etc/jenkins_jobs.ini-sample
   :language: ini

job_builder section
^^^^^^^^^^^^^^^^^^^

**ignore_cache**
  (Optional) If set to True, Jenkins Job Builder won't use any cache.

**keep_descriptions**
  By default `jenkins-jobs` will overwrite the jobs descriptions even if no
  description has been defined explicitly.
  When this option is set to True, that behavior changes and it will only
  overwrite the description if you specified it in the yaml. False by default.

**include_path**
  (Optional) Can be set to a ':' delimited list of paths, which jenkins
  job builder will search for any files specified by the custom application
  yaml tags 'include', 'include-raw' and 'include-raw-escape'.

**recursive**
  (Optional) If set to True, jenkins job builder will search for job
  definition files recursively.

**exclude**
  (Optional) If set to a list of values separated by ':', these paths will be
  excluded from the list of paths to be processed when searching recursively.
  Values containing no ``/`` will be matched against directory names at all
  levels, those starting with ``/`` will be considered absolute, while others
  containing a ``/`` somewhere other than the start of the value will be
  considered relative to the starting path.

**allow_duplicates**
  (Optional) By default `jenkins-jobs` will abort when a duplicate macro,
  template, job-group or job name is encountered as it cannot establish the
  correct one to use. When this option is set to True, only a warning is
  emitted.

**allow_empty_variables**
  (Optional) When expanding strings, by default `jenkins-jobs` will raise an
  exception if there's a key in the string, that has not been declared in the
  input YAML files. Setting this option to True will replace it with the empty
  string, allowing you to use those strings without having to define all the
  keys it might be using.

**print_job_urls**
  (Optional) If set to True it will print full jobs urls while updating jobs,
  so user can be sure which instance was updated. User may click the link to
  go directly to that job. False by default.

**retain_anchors**
  (Optional) If set to True, YAML anchors will be retained across files,
  allowing jobs to be composed from bits of YAML defined in separate files.
  Note this means that the order of processing files matters - `jenkins-jobs`
  loads files in alphabetical order (all files in a dir are loaded before any
  files in subdirs). For example, if your anchors are in a file named `foo.yml`
  they will be accessible in `qux.yml` but not in `bar.yml`. They will also be
  accessible in `mydir/bar.yml` and `mydir/qux.yml`. False by default.

**update**
  (Optional) If set, allows the user to specify if only "jobs" or "views"
  (or "all") are updated. Users can override the setting here by passing
  ``--jobs-only`` or ``--views-only`` CLI options.
  (Valid options: jobs, views, all)

jenkins section
^^^^^^^^^^^^^^^

**user**
  This should be the name of a user previously defined in Jenkins.
  Appropriate user permissions must be set under the Jenkins security
  matrix: under the ``Global`` group of permissions, check ``Read``,
  then under the ``Job`` group of permissions, check ``Create``,
  ``Delete``, ``Configure`` and finally ``Read``.

**password**
  The API token for the user specified.  You can get this through the
  Jenkins management interface under ``People`` -> username ->
  ``Configure`` and then click the ``Show API Token`` button.

**url**
  The base URL for your Jenkins installation.

**timeout**
  (Optional) The connection timeout (in seconds) to the Jenkins server.
  By default this is set to the system configured socket timeout.

**query_plugins_info**
  Whether to query the Jenkins instance for plugin info. If no configuration
  files are found (either in the default paths or given through the
  command-line), `jenkins-jobs` will skip querying for plugin information. True
  by default.


hipchat section
^^^^^^^^^^^^^^^

**send-as**
  This is the hipchat user name that will be used when sending notifications.

**authtoken**
  The API token necessary to send messages to hipchat.  This can be generated in
  the hipchat web interface by a user with administrative access for your
  organization. This authtoken is set for each job individually; the
  JJB Hipchat Plugin does not currently support setting different tokens for
  different projects, so the token you use will have to be scoped such that it
  can be used for any room your jobs might be configured to notify. For more
  information on this topic, please see the `Hipchat API Documentation`__

__ https://developer.atlassian.com/server/hipchat/about-the-hipchat-rest-api/


stash section
^^^^^^^^^^^^^^^^^^^^^^^

**username**
  This is the stash user name that will be used to connect to stash
  when using the stash publisher plugin and not defining it in the
  yaml part.

**password**
  This is the related password that will be used with the stash username
  when using the stash publisher plugin and not defining it in the
  yaml part.


__future__ section
^^^^^^^^^^^^^^^^^^

This section is to control enabling of beta features or behaviour changes that
deviate from previously released behaviour in ways that may require effort to
convert existing JJB configs to adopt. This essentially will act as a method
to share these new behaviours while under active development so they can be
changed ahead of releases.

**param_order_from_yaml**
  Used to switch on using the order of the parameters are defined in yaml to
  control the order of corresponding XML elements being written out. This is
  intended as a global flag and can affect multiple modules.


Running
-------

After it's installed and configured, you can invoke Jenkins Job
Builder by running ``jenkins-jobs``.  You won't be able to do
anything useful just yet without a configuration; that is
discussed in the next section.

Test Mode
^^^^^^^^^
Once you have a configuration defined, you can run the job builder in test mode.

If you want to run a simple test with just a single YAML job definition file
and see the XML output on stdout::

  jenkins-jobs test /path/to/foo.yaml

You can also pass JJB a directory containing multiple job definition files::

  jenkins-jobs test /path/to/defs -o /path/to/output

which will write XML files to the output directory for all of the jobs
defined in the defs directory.

If you run::

  jenkins-jobs test /path/to/defs -o /path/to/output --config-xml

the output directory will contain config.xml files similar to the
internal storage format of Jenkins.  This might allow you to more
easily compare the output to an existing Jenkins installation.

.. _updating-jobs:

Updating Jobs
^^^^^^^^^^^^^
When you're satisfied with the generated XML from the test, you can run::

  jenkins-jobs update /path/to/defs

which will upload the job and view definitions to Jenkins if needed.  Jenkins
Job Builder maintains, for each host, a cache [#f1]_ of previously configured
jobs and views, so that you can run that command as often as you like, and it
will only update the jobs configurations in Jenkins if the defined definitions
have changed since the last time it was run.  Note: if you modify a job
directly in Jenkins, jenkins-jobs will not know about it and will not
update it.

To update a specific list of jobs/views, simply pass the job/view names as
additional arguments after the job definition path. To update Foo1 and Foo2
run::

  jenkins-jobs update /path/to/defs Foo1 Foo2

You can also enable the parallel execution of the program passing the workers
option with a value of 0, 2, or higher. Use 0 to run as many workers as cores
in the host that runs it, and 2 or higher to specify the number of workers to
use::

  jenkins-jobs update --workers 0 /path/to/defs

To update only views or only jobs, simply add the argument
--views-only or --jobs-only after the command::

  jenkins-jobs update --views-only Foo-view
  jenkins-jobs update --jobs-only Foo-job


Passing Multiple Paths
^^^^^^^^^^^^^^^^^^^^^^
It is possible to pass multiple paths to JJB using colons as a path separator on
\*nix systems and semi-colons on Windows systems. For example::

  jenkins-jobs test /path/to/global:/path/to/instance:/path/to/instance/project

This helps when structuring directory layouts as you may selectively include
directories in different ways to suit different needs. If you maintain multiple
Jenkins instances suited to various needs you may want to share configuration
between those instances (global). Furthermore, there may be various ways you
would like to structure jobs within a given instance.

Recursive Searching of Paths
----------------------------

In addition to passing multiple paths to JJB it is also possible to enable
recursive searching to process all yaml files in the tree beneath each path.
For example::

  For a tree:
    /path/
      to/
        defs/
          ci_jobs/
          release_jobs/
        globals/
          macros/
          templates/

  jenkins-jobs update -r /path/to/defs:/path/to/globals

JJB will search defs/ci_jobs, defs/release_jobs, globals/macros and
globals/templates in addition to the defs and globals trees.

Excluding Paths
---------------

To allow a complex tree of jobs where some jobs are managed differently without
needing to explicitly provide each path, the recursive path processing supports
excluding paths based on absolute paths, relative paths and patterns. For
example::

  For a tree:
    /path/
      to/
        defs/
          ci_jobs/
            manual/
          release_jobs/
            manual/
          qa_jobs/
        globals/
          macros/
          templates/
          special/

  jenkins-jobs update -r -x man*:./qa_jobs -x /path/to/defs/globals/special \
    /path/to/defs:/path/to/globals

JJB will search the given paths, ignoring the directories qa_jobs,
ci_jobs/manual, release_jobs/manual, and globals/special when
building the list of yaml files to be processed. Absolute paths
are denoted by starting from the root, relative by containing
the path separator, and patterns by having neither.
Patterns use simple shell globing to match directories.

Deleting Jobs/Views
^^^^^^^^^^^^^^^^^^^
Jenkins Job Builder supports deleting jobs and views from Jenkins.

To delete a specific job::

  jenkins-jobs delete Foo1

To delete a list of jobs or views, simply pass them as additional
arguments after the command::

  jenkins-jobs delete Foo1 Foo2

To delete only views or only jobs, simply add the argument
--views-only or --jobs-only after the command::

  jenkins-jobs delete --views-only Foo1
  jenkins-jobs delete --jobs-only Foo1

The ``update`` command includes a ``delete-old`` option to remove obsolete
jobs::

  jenkins-jobs update --delete-old /path/to/defs

Obsolete jobs are jobs once managed by JJB (as distinguished by a special
comment that JJB appends to their description), that were not generated in this
JJB run.

There is also a command to delete **all** jobs and/or views.
**WARNING**: Use with caution.

To delete **all** jobs and views::

  jenkins-jobs delete-all

TO delete **all** jobs::

  jenkins-jobs delete-all --jobs-only

To delete **all** views::

  jenkins-jobs delete-all --views-only

Globbed Parameters
^^^^^^^^^^^^^^^^^^
Jenkins job builder supports globbed parameters to identify jobs from a set of
definition files.  This feature only supports JJB managed jobs.

To update jobs/views that only have 'foo' in their name::

  jenkins-jobs update ./myjobs \*foo\*

To delete jobs/views that only have 'foo' in their name::

  jenkins-jobs delete --path ./myjobs \*foo\*

Providing Plugins Info
^^^^^^^^^^^^^^^^^^^^^^
With Jenkins LTS 1.651.1+ retrieving plugins info became a secure feature and
now requires Administrator rights to use [#f2]. This causes JJB to no longer be
able to work in situations where a user wants to publish jobs to Jenkins but is
not able to receive the Administrator permissions. In this case we can provide
a plugins_info.yaml file containing the plugin versions data needed by JJB to
parse the job templates.

To generate a plugins info, using an account with Administrator rights:

  jenkins-jobs get-plugins-info -o plugins_info.yaml

To run JJB update using the plugins_info.yaml:

  jenkins-jobs update -p plugins_info.yaml ./myjobs

.. _command-reference:

Command Reference
^^^^^^^^^^^^^^^^^
.. program-output:: jenkins-jobs --help
.. program-output:: jenkins-jobs test --help
.. program-output:: jenkins-jobs update --help
.. program-output:: jenkins-jobs delete-all --help
.. program-output:: jenkins-jobs delete --help
.. program-output:: jenkins-jobs get-plugins-info --help

.. rubric:: Footnotes
.. [#f1] The cache default location is at ``~/.cache/jenkins_jobs``, which
         can be overridden by setting the ``XDG_CACHE_HOME`` environment
         variable.
.. [#f2] Jenkins Security Advisory affecting plugins info retrieval
         https://wiki.jenkins.io/display/SECURITY/Jenkins+Security+Advisory+2016-05-11
