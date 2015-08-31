# Google Cloud Platform plugin for Cloudera Director
The Cloudera Director Google Plugin is an implementation of the [Cloudera Director Service Provider Interface](https://github.com/cloudera/director-spi) for the [Google Cloud Platform](https://cloud.google.com). It presently supports provisioning Google Compute Engine resources and will soon expand to include support for provisioning Google Cloud SQL resources.

These instructions describe how to configure and exercise the Cloudera Director Google Plugin.

Prior to using this plugin, you will need to have a Google Cloud Platform project and have obtained [Service Account](https://cloud.google.com/compute/docs/authentication#general) credentials for use in programmatically accessing it.

## Configure your Google Cloud Platform project
Note: These instructions assume you have a Google Cloud Platform project with billing enabled.

If you are running Cloudera Director on a Google Compute Engine VM instance within the Google Cloud Platform project that will contain your Cloudera clusters, you can have the plugin automatically retrieve credentials from the environment. You simply need to ensure that the instance was created with [Read-write access to Compute Engine methods](https://cloud.google.com/compute/docs/authentication) enabled.

If you are running Cloudera Director outside of Google Compute Engine, or in a different Google Cloud Platform project, you will need to obtain a JSON key:
* Point your browser at your [Google Developers Console](https://console.developers.google.com/).
* Navigate to: Projects->{your-project-name}->APIs & auth->APIs
* Enable the Google Compute Engine API.
* Navigate to: Projects->{your-project-name}->APIs & auth->Credentials.
* Select "Add credentials->Service account".
* Ensure "JSON" option is enabled.
* Click "Create" button.
* Dismiss "New public/private key pair" popup with "OK" button.
* Note the location of your newly-downloaded .json file.

## Prerequisites
* You've either followed the instructions above to download a json key for use in authenticating to your Google Cloud Platform project, or you are running Cloudera Director on a Google Compute Engine VM instance within the same Google Cloud Platform project.
* Maven is installed.

## Configure the Tests
The `GCP_PROJECT_ID` env variable must be set to your Project ID, not the Project Name. (The Project Name is a user-friendly name that you assign when you create your project. Might be something like `"My Cloudera Project"`. The Project ID is either user-assigned, or can be auto-generated by Google Developers Console, and is typically used to refer to a project in requests. Often looks like `"my-cloudera-project"`.)

If using a json key for authentication, the `JSON_KEY_PATH` env variable must contain the full absolute path to the downloaded .json file.

The `SSH_PUBLIC_KEY_PATH` env variable must contain the full absolute path to the ssh public key whose private counterpart you intend to use to ssh into the newly-provisioned instances.

The `SSH_USER_NAME` env variable must contain the name of the user you intend to use to ssh into the newly-provisioned instances.

## Run the Tests
```bash
$ # Without json key (using credentials obtained automatically from the Google Compute Engine environment):
$ cd director-google-plugin
$ mvn clean install -DGCP_PROJECT_ID=$GCP_PROJECT_ID -DSSH_PUBLIC_KEY_PATH=$SSH_PUBLIC_KEY_PATH -DSSH_USER_NAME=$SSH_USER_NAME

$ # With json key:
$ cd director-google-plugin
$ mvn clean install -DGCP_PROJECT_ID=$GCP_PROJECT_ID -DSSH_PUBLIC_KEY_PATH=$SSH_PUBLIC_KEY_PATH -DSSH_USER_NAME=$SSH_USER_NAME -DJSON_KEY_PATH=$JSON_KEY_PATH
```

There are various unit tests in the test suite (some of them require 'live' access to your Google Cloud Platform project) and one [integration test](https://github.com/cloudera/director-google-plugin/blob/master/tests/src/test/java/com/cloudera/director/google/compute/GoogleComputeProviderFullCycleTest.java). The integration test will provision 2 new instances (one centos6, one rhel6) with attached local SSD data disks, attempt to provision each again (to verify idempotency), and then tear down the instances. It will poll for status and verify the operation results at each stage.

If you set the optional property HALT_AFTER_ALLOCATION to true, the integration test will leave both instances running:
```bash
$ # Without json key (using credentials obtained automatically from the Google Compute Engine environment):
$ mvn clean install -DGCP_PROJECT_ID=$GCP_PROJECT_ID -DSSH_PUBLIC_KEY_PATH=$SSH_PUBLIC_KEY_PATH -DSSH_USER_NAME=$SSH_USER_NAME -DHALT_AFTER_ALLOCATION=true

$ # With json key:
$ mvn clean install -DGCP_PROJECT_ID=$GCP_PROJECT_ID -DSSH_PUBLIC_KEY_PATH=$SSH_PUBLIC_KEY_PATH -DSSH_USER_NAME=$SSH_USER_NAME -DJSON_KEY_PATH=$JSON_KEY_PATH -DHALT_AFTER_ALLOCATION=true
```

You can then access the Google Developers Console to view the resulting instances by navigating to: Projects->{your-project-name}->Compute->Compute Engine->VM instances.

To verify that your ssh key was properly set, you can retrieve the External IP of one of your instances from the VM instances view and ssh in (assuming you set the External IP into the EXTERNAL_IP env variable):
```bash
$ ssh $SSH_USER_NAME@$EXTERNAL_IP
```

## Install the Plugin in Cloudera Director
There are instructions on plugin installation in the Cloudera Director Service Provider Interface [documentation](https://github.com/cloudera/director-spi#installing-the-plugin).

## Copyright and License
Copyright © 2015 Google. Licensed under the Apache License.

See [LICENSE.txt](https://github.com/cloudera/director-google-plugin/blob/master/LICENSE) for more information.
