# cf-deployment

## Introduction

This is cf-deployment pipeline for Azure. The pipeline trigger on new commit on [release-candidate branch in cf-deployment repo](https://github.com/cloudfoundry/cf-deployment/tree/release-candidate). If triggered, the pipeline will use [bbl](https://github.com/cloudfoundry/bosh-bootloader) to setup a bosh director, then deploy cloud foundry using bosh, run cats on cloud foundry. If cats passed, all allocated resources will be deleted safely; if cats failed, a notification email and a slack message will be send to developer.

## Structure

In the pipeline, only one job `job-create-certs` is not from [cf-deployment-concourse-tasks](https://github.com/cloudfoundry/cf-deployment-concourse-tasks) repo. This job is used for creating cert, which will be used in the next job `job-run-bbl-up` . Each of other jobs contains exactly one task in  [cf-deployment-concourse-tasks](https://github.com/cloudfoundry/cf-deployment-concourse-tasks) repo, you can get detailed information of them in the [doc](https://github.com/cloudfoundry/cf-deployment-concourse-tasks#tasks).

## Prerequisite

- A concourse environment for running pipeline. You can create concourse using Azure [template](https://github.com/Azure/azure-quickstart-templates/tree/master/concourse-ci).
- A private git repo, this repo is used to store and transfer sensitive informations. For example, the private key of the bosh director, the admin password of cloud foundry will be stored in this repo.
- An Azure service principal. It is used by bosh director to create resources. You can learn how to create service princiapl [here](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2Fazure%2Fazure-resource-manager%2Ftoc.json&view=azure-cli-latest).

## Parameters

There are three parameters files: `credentials.yml` , `email-credential.yml` , `slack-credential.yml` . You should fill all of them. The following is detailed description of parameters:

- In`credential.yml`: actually, these are parameters in [cf-deployment-concourse-tasks](https://github.com/cloudfoundry/cf-deployment-concourse-tasks). You can find the meaning of these parameters in [cf-deployment-concourse-tasks](https://github.com/cloudfoundry/cf-deployment-concourse-tasks) repo. Here is an example:

  ```yaml
  ---
  # Used globally, git email and username used to push to the repo.
  GIT_COMMIT_EMAIL: my_email@outlook.com
  GIT_COMMIT_USERNAME: my_username

  # This is the private key of the store repo.
  BBLSTATE_REPO_URI: git@github.com:username/repo.git
  BBLSTATE_BRANCH: master
  BBLSTATE_PRIVATE_KEY: |
      -----BEGIN RSA PRIVATE KEY-----
      ...
      The private key with write access of your repo.
      ...
      -----END RSA PRIVATE KEY-----

  # Used in bbl-up
  BBL_AZURE_CLIENT_ID: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  BBL_AZURE_CLIENT_SECRET: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  BBL_AZURE_TENANT_ID: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  BBL_AZURE_SUBSCRIPTION_ID: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  BBL_AZURE_REGION: eastasia
  BBL_LB_CERT: CERT_FILE_NAME
  BBL_LB_KEY: KEY_FILE_NAME
  LB_DOMAIN: mydomain.com

  # Used in bosh-deploy
  SYSTEM_DOMAIN: mydomain.com

  # Used in run-cats, usually you don't change it
  CONFIG_FILE_PATH: cats_integration_config.json
  ```

  ​

- In `email-credential.yml`: these are parameters used to send email, you should change them to your own, for example:

  ```yaml
  ---
  smtp-host: smtp-mail.outlook.com
  smtp-port: "587"
  smtp-username: my_username@outlook.com
  smtp-password: my_password
  email-from: my_username@outlook.com
  email-to: developer_email_address@example.com
  SUBJECT: "CATs Failed"
  BODY: "CATs Failed. Please check your pipeline for detailed information" 
  ```

  You may need to get `smtp-host`, `smtp-port`, `smtp-username`, `smtp-password` from your own  email service provider.

- In `slack-credential.yml`: these are parameters used to send slack message, you should change them to your own, for example:

  ```yaml
  ---
  # Slack webhook used to send message
  SLACK_WEBHOOK: https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX

  # Message send by webhook
  SLACK_TEXT: "Pipeline failed"
  ```

  You can learn how to create a slack webhook [here](https://get.slack.help/hc/en-us/articles/115005265063-Incoming-WebHooks-for-Slack). 

## Usage

This pipeline is automatically triggered, unless CATs fails or unpredictable error occurs, you don't need to manipulate it manually. 

To setup this pipeline, use  `fly -t your_target_name sp -c pipeline.yml -l credential.yml -l email-credential.yml -l slack-credential.yml -p name_of_the_pipeline`.
