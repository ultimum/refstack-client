===============
RefStack Client
===============

Overview
########

``refstack-client`` is a command line utility that allows you to execute Tempest
test runs based on configurations you specify.  When finished running Tempest
it can send the passed test data to a RefStack API server.

Environment setup
#################

We've created an "easy button" for Ubuntu, Centos, RHEL and openSUSE.

1. Make sure you have ``git`` installed
2. Get the refstack client: ``git clone https://opendev.org/osf/refstack-client.git``
3. Go into the ``refstack-client`` directory: ``cd refstack-client``
4. Run the "easy button" setup: ``./setup_env``

   **Options:**

   a. -c option allows to specify SHA of commit or branch in Tempest repository
   which will be installed.

   b. -t option allows to specify tag in Tempest repository which will be installed.
   For example: execute ``./setup_env -t tags/3`` to install Tempest tag-3.
   By default, Tempest will be installed from commit
   1d500e79156ada6bc6fdb628ed1da0efd4121f6a (Oct, 2019).

   c. -p option allows to specify python version - python 2.7 (-p 2), 3.6 (-p 3)
   or any specific one by -p X.X.X. Default to python 3.6.

   d. -q option makes ``refstack-client`` run quitely - if ``.tempest``
   directory exists ``refstack-client`` is considered as installed.

   e. -s option makes ``refstack-client`` use ``python-tempestconf`` from the
   given source (path) - used when running f.e. in Zuul.

Usage
#####

1. Prepare a tempest configuration file (or let ``refstack-client`` generate it
   for you, see step #4) that is customized to your cloud environment.
   Samples of minimal Tempest configurations are provided in the ``etc``
   directory in ``tempest.conf.sample`` and ``accounts.yaml.sample``.
   Note that these samples will likely need changes or additional information
   to work with your cloud.

   **Note**: Use Tempest Pre-Provisioned credentials_ to provide user test
   accounts.

.. _credentials: https://docs.openstack.org/tempest/latest/configuration.html#pre-provisioned-credentials

2. Go into the ``refstack-client`` directory::

       cd ~/refstack-client

3. Source to use the correct Python environment::

       source .venv/bin/activate

4. (optional) Generate tempest.conf using ``refstack-client``::

       refstack-client config --use-test-accounts <path to account file>

   The above command will create the tempest.conf in `etc` folder.

   Note: If account file is not available, then:
   * Source the keystonerc file containing cloud credentials and run::

         refstack-client config

     It will create accounts.yaml and temepst.conf file in `etc` folder.

5. Validate your setup by running a short test::

       refstack-client test \
         -c <Path of the tempest configuration file to use> -v -- \
         --regex tempest.api.identity.v3.test_tokens.TokensV3Test.test_create_token

6. Run tests.

   To run the entire API test set::

       refstack-client test -c <Path of the tempest configuration file to use> -v

   To run only those tests specified in an OpenStack Powered (TM) Guideline::

       refstack-client test -c <Path of the tempest configuration file to use> -v --test-list <Absolute path  of test list>

   For example::

       refstack-client test \
         -c ~/tempest.conf -v \
         --test-list "https://refstack.openstack.org/api/v1/guidelines/2020.11/tests?target=platform&type=required&alias=true&flag=false"

   This will run only the test cases required by the 2020.11 guidelines under
   Platform OpenStack Marketing Program that have not been flagged. More about
   the marketing programs at `Interop and OpenStack Marketing Programs`_.

   For example tests under the compute program are available:
   https://refstack.openstack.org/api/v1/guidelines/2020.11/tests?target=compute&type=required&alias=true&flag=false
   Tests of add-on programs can be found similarly, f.e. tests under dns program:
   https://refstack.openstack.org/api/v1/guidelines/dns.2020.11/tests?target=dns&type=required&alias=true&flag=false
   or tests under orchestration program:
   https://refstack.openstack.org/api/v1/guidelines/orchestration.2020.11/tests?target=orchestration&type=required&alias=true&flag=false

   **Note:**

   a. Adding the ``-v`` option will show the Tempest test result output.
   b. Adding the ``--upload`` option will have your test results be uploaded to the
      default RefStack API server or the server specified by ``--url``.
   c. Adding the ``--test-list`` option will allow you to specify the file path or URL of
      a test list text file. This test list should contain specific test cases that
      should be tested. Tests lists passed in using this argument will be normalized
      with the current Tempest environment to eliminate any attribute mismatches.
   d. Adding the ``--url`` option will allow you to change where test results should
      be uploaded.
   e. Adding the ``-r`` option with a string will prefix the JSON result file with the
      given string (e.g. ``-r my-test`` will yield a result file like
      'my-test-0.json').
   f. Adding ``--`` enables you to pass arbitrary arguments to tempest run.
      After the first ``--``, all other subsequent arguments will be passed to
      tempest run as is. This is mainly used for quick verification of the
      target test cases. (e.g. ``-- --regex tempest.api.identity.v2.test_token``)
   g. If you have provisioned multiple user/project accounts you can run parallel
      test execution by enabling the ``--parallel`` flag.

   Use ``refstack-client test --help`` for the full list of arguments.

6. Upload your results.

   If you previously ran a test with ``refstack-client`` without the ``--upload``
   option, you can later upload your results to a RefStack API server
   with your digital signature. By default, the results are private and you can
   decide to share or delete the results later.

   Following is the command to upload your result::

       refstack-client upload <Path of results file> -i <path-to-private-key>

   The results file is a JSON file generated by ``refstack-client`` when a test has
   completed. This is saved in .tempest/.stestr. When you use the
   ``upload`` command, you can also override the RefStack API server uploaded to
   with the ``--url`` option.

   Alternatively, you can use the ``upload-subunit`` command to upload results
   using an existing subunit file. This requires that you pass in the Keystone
   endpoint URL for the cloud that was tested to generate the subunit data::

       refstack-client upload-subunit \
         --keystone-endpoint http://some.url:5000/v3 <Path of subunit file> \
         -i <path-to-private-key>

   Intructions for uploading data with signature can be found at
   https://opendev.org/osf/refstack/src/branch/master/doc/source/uploading_private_results.rst

7. View uploaded test set.

   You can list previously uploaded data from a RefStack API server by using
   the following command::

       refstack-client list --url <URL of the RefStack API server> -i <path to private key>

   Alternatively, if you uploaded the results to the official RefStack_ server
   you can view them by using RefStack_ page where all uploaded results
   associated with the particular account (the account private key used to
   upload the results belongs to) will be shown and may be further managed.


Tempest hacking
###############

By default, ``refstack-client`` installs Tempest into the ``.tempest`` directory.
If you're interested in working with Tempest directly for debugging or
configuration, you can activate a working Tempest environment by
switching to that directory and using the installed dependencies.

1. ``cd .tempest``
2. ``source ./.venv/bin/activate``
   and run tests manually with ``tempest run``.

This will make the entire Tempest environment available for you to run,
including ``tempest run``. More about Tempest can be found at its documentation_.

.. _documentation: https://docs.openstack.org/tempest/latest/


Interop and OpenStack Marketing Programs
########################################

The tests ``refstack-client`` runs are defined within interop_ repository
and divided into several OpenStack Marketing Programs, the list of the programs
can be found at RefStack_ page.

.. _interop: https://opendev.org/osf/interop
.. _RefStack: https://refstack.openstack.org/#/


ansible-role-refstack-client
############################

We have created an ansible role called ansible-role-refstack-client_ in order
to simplify and automate running of ``refstack-client``. The role can be easily
integrated to an automation machinery - f.e. we use the role for running
``refstack-client`` on a devstack_ environment in Zuul where we run tests of
every OpenStack Marketing Program of the current guideline. The latest builds
can be found here__.

.. _ansible-role-refstack-client: https://opendev.org/x/ansible-role-refstack-client
.. _devstack: https://opendev.org/openstack/devstack/
.. __builds: https://zuul.openstack.org/builds?project=x%2Fansible-role-refstack-client
