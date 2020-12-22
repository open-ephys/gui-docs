.. _distributingplugins:
.. role:: raw-html-m2r(raw)
   :format: html

Distributing plugins
=====================

Once a plugin has been :ref:`created <creatinganewplugin>`, it can be made available through the Open Ephys Plugin Installer by following the instructions on this page. Any plugins distributed this way must first be vetted and polished by the Open Ephys core team. If you have a plugin you believe is ready to be shared with the community, please get in touch!

.. warning:: Some of the steps on this page require private information and login credentials contained in a private GitHub repository, :code:`open-ephys-plugins/open-ephys-plugins-ci`.

Setting up the plugin repository
#################################

1. Fork the plugin of interest to the :code:`open-ephys-plugins` GitHub account (requires login).

2. Add :code:`bintrayUsername` and :code:`bintrayApiKey` to GitHub secrets under repository settings.

3. Add GitHub workflows for macOS, Linux, and Windows (exampes available in the :code:`open-ephys-plugins-ci` repository)

.. note:: If the plugin has dependencies that are a part of the plugin repository, refer to the `neuropixels-pxi <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ plugin workflows to understand the proper way to package dependencies inside the plugin's zip file.

Setting up the Bintray repository
#################################

1. Open the :code:`open_ephys_plugins_ci.py` script.

2. Modify the script with the plugin-specific information.

3. Run the script.

4. Login to the Bintray account.

5. Verify the repository and its packages have been created. 

.. note:: Be sure the labels for the repository have been set correctly. The first label should always be the plugin display name that should appear in Plugin Installer. Remaining labels specify the type of plugin, e.g. Source, Filter, or Sink.

Configuring external dependencies (if any)
##############################################

If the plugin needs any external dependency that requires its own repository to build, you'll have to follow all the steps mentioned above to set up the GitHub and Bintray repositories. **Importantly,** while setting up the Bintray repository using the script, make sure you add "Dependency" as the second label (type of plugin) in the script. 

Once the external dependency is setup, go to the plugin's repository that needs this dependency. For each platform package, go to the "Custom Attributes" menu and add the dependency *repository* name to the "Name" field and the dependency *package* name for the platform as the "Value".

Releasing the plugin
#################################

* For each new release, tag the latest commit on GitHub with the version number. Versions should follow the `semantic versioning <https://semver.org/>`_ scheme. (Do not add a 'v' in front of the version number).

* After pushing the tag to the repository, it should run the GitHub Actions workflows across all the platforms which should build the plugin, package it, and publish it to Bintray.

* Once the workflows run successfully, the plugins (or its latest version) should be available to download from the GUI's Plugin Installer.

.. note:: If there is an external dependency which also needs to be updated and released, do that prior to releasing the plugin that depends on it.