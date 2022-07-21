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

2. Go to :code:`open-ephys-plugins` organization settings and navigate to Secrets->Actions.

   .. image:: ../_static/images/developerguide/artifactory-secret.png
      :alt: Organization secret settings
      :target: ../_static/images/developerguide/artifactory-secret.png

3. Update the :code:`ARTIFACTORYAPIKEY` settings by clicking on Update and then going to Repository access settings.

4. Select and enable your plugin repository from the list of organization repositories, and click on update selection.   

Setting up the Artifactory repository
######################################

1. Login to the `openephys.jfrog.io <https://openephys.jfrog.io/>`_ account.

2. Go to Administration settings and open Repositories from the left sidebar.

   .. image:: ../_static/images/developerguide/jfrog-repository-settings.png
      :alt: JFrog repository settings
      :target: ../_static/images/developerguide/jfrog-repository-settings.png

3. Click on "Add Repositories" on the top-right corner and create a "Local Repository". Select the package type as "Generic".

4. Now, specify the name of the plugin in the :code:`Repository Key` field.

   .. note:: The name of the repository should have the :code:`-plugin` string appended. So, if your plugin name is Online PSTH, then the repository name should be :code:`OnlinePSTH-plugin`.

   .. image:: ../_static/images/developerguide/create-artifactory-repo.png
      :alt: Create Artifactory Repository
      :target: ../_static/images/developerguide/create-artifactory-repo.png

5. Enter the plugin description in the :code:`Public Description` field.

6. Click on :code:`Create Local Repository`. That should create the plugin Artifactory repository for you.

7. Now, go to the Artifactory homepage and click on **Artifactory > Artifacts**. 

8. Navigate to the repository you just created.

9. Click on properties, and add the following properties to the repository: 

   .. csv-table:: Properties for Artifactory repository
      :header: "Property Name", "Property Value"
      :widths: 30, 120

      "**Name**", "The actual name of the plugin as specified in the :code:`OpenEphysLib.cpp` file"
      "**Type**", "Source, Filter, Sink, RecordEngine, or CommonLib"
      "**Developers**", "Name(s) of the plugin developer(s)"
      "**Docs**", "Link to the plugin documentation page on"
      "**Dependencies**", "*(Optional)* Names of external plugin dependencies (if any) that have their own repository"

   .. image:: ../_static/images/developerguide/artifactory-repo-properties.png
      :alt: Create Artifactory Repository
      :target: ../_static/images/developerguide/artifactory-repo-properties.png

Configuring external dependencies (if any)
##############################################

If the plugin has any external dependency that requires its own repository to be built and deployed, you'll have to follow all the steps mentioned above to set up the GitHub and Artifactory repositories. **Importantly,** while setting up the Artifactory repository properties, make sure you add "Type" property with "CommonLib" as it's value. There's no need to specify any other properties.

Once the external dependency is setup, go to the plugin's Artifactory repository that needs this dependency. Add the "Dependencies" property and the dependency repository name without the appended "-plugin" string as the value.

Releasing the plugin
#################################

* For each new release, tag the latest commit on GitHub with the version number. Versions should follow the `semantic versioning <https://semver.org/>`_ scheme. (Do not add a 'v' in front of the version number).

* Add GitHub Actions workflows for macOS, Linux, and Windows (examples available in the private `open-ephys-plugins-ci <https://github.com/open-ephys-plugins/open-ephys-plugins-ci>`__ repository)

.. note:: If the plugin has dependencies that are a part of the plugin repository, refer to the `neuropixels-pxi <https://github.com/open-ephys-plugins/neuropixels-pxi>`__ plugin workflows to understand the proper way to package dependencies inside the plugin's zip file.

* After pushing the workflows and the tag to the Github repository, it should run the GitHub Actions workflows across all the platforms which should build the plugin, package it, and publish it to Artifactory.

* Once the workflows run successfully, the plugins (or its latest version) should be available to download from the GUI's Plugin Installer.

.. note:: If there is an external dependency which also needs to be updated and released, do that prior to releasing the plugin that depends on it.