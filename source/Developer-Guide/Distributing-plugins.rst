.. _distributingplugins:
.. role:: raw-html-m2r(raw)
   :format: html

Distributing plugins
=====================

This page contains instructions for preparing plugins for distribution through the Open Ephys Plugin Installer. Currently, only members of the core Open Ephys team are allowed to do this. Refer to the :code:`open_ephys_plugins_ci` repository to obtain the required files & information.

Setting up plugin repository
#################################

1. Fork the plugin to the :code:`open-ephys-plugins` account (requires logging in to the account).

2. Add bintrayUsername and bintrayApiKey to GitHub secrets under repository settings.

3. Add GitHub workflows

.. note:: If your plugin has dependencies that are a part of the plugin repository, refer to neuropixel-pxi's workflows for packaging the dependencies inside the plugin's zip file.

Setting up Bintray repository
#################################

1. Open :code:`open_ephys_plugins_ci.py` script :sup:`*`

2. Modify the script to add your information as mentioned in the script.

3. Run the script

4. Login to the Bintray account :sup:`*`

5. Verify the your repository and its packages have been created. 

.. note:: Make sure the labels for the repository are setup correctly. The first label should always be plugin display name that should appear in Plugin Installer. Remaining labels are the type of plugin it is: Source, Filter, or Sink.

External dependencies (if any)
#################################

If your plugin needs any external dependency that requires its own repository to build, you'll have to follow all the steps mentioned above to setup GitHub and Bintray repositories.

.. note:: While setting up the Bintray repository using the script, make sure you add "Dependency" as the second label (type of plugin) in the script. 

Once the external dependency is setup, go to the plugin's repository that needs this dependency and for each platform package, go to the "Custom Attributes" menu and add the dependency *repository* name to the "Name" field and the dependency *package* name for the platform as the "Value".

Releasing the plugin
#################################

* For each new release, tag the latest commit on GitHub with the version number. Versions should follow the `semantic versioning <https://semver.org/>`_ scheme. (Do not add a 'v' in front of the version number).

* After pushing the tag to the repository, it should run GitHub Actions workflows across all the platforms which should build the plugin, package it, and publish it to Bintray.

* Once the workflows run successfully, the plugins (or its latest version) should be available to download from Plugin Installer.

.. note:: If there is an external dependency which also needs to be updated and released, do that first before releasing the plugin that depends on it.