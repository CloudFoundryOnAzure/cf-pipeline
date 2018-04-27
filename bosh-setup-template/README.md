# bosh-setup-template

## Introduction

This pipeline can check new version of manifests in [cf-deployment](https://github.com/cloudfoundry/cf-deployment.git). If new version is found, the pipeline will deploy a cf using new manifest, then run [CATs](https://github.com/cloudfoundry/cf-acceptance-tests/) on it. If CATs passed, new version of the manifest will bump to developer's repo, and a notification email will be sent to developer.

## Structure

The functionality of every job:
- [check-new-version](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/lib/check-new-version): this job compares manifest in [cf-deployment](https://github.com/cloudfoundry/cf-deployment.git) and [Azure template repo](https://github.com/CloudFoundryOnAzure/cf-pipeline/blob/master/bosh-setup-template/credential.yml#L22-L24), if any new manifest is found in cf-deployment, the new manifest will bump to the template repo. If no new manifest is found, the job will fail and later job will not be triggered.
- [deploy-cf](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/lib/deploy-cf): this job uses new manifest created in job `check-new-version` and template in [Azure template repo](https://github.com/CloudFoundryOnAzure/cf-pipeline/blob/master/bosh-setup-template/credential.yml#L22-L24) to deploy a cloud foundry. And the credential to login cloud foundry will be uploaded to [config repo](https://github.com/CloudFoundryOnAzure/cf-pipeline/blob/master/bosh-setup-template/credential.yml#L16-L19).
- [login-cf](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/lib/login-cf): due to the process of creating cf is asynchronous on Azure, this job will continuously try to login cf. The job will pass when it successfully login cf, which indicates the creation of cf is done.
- [run-cats](https://github.com/cloudfoundry/cf-deployment-concourse-tasks#run-cats): this job uses offical run-cats task, it runs CATs on the cf.
- [delete-resource-group](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/lib/delete-resource-group): this job deletes all resource created in previous jobs and sends a notification email to the developer.

## Prerequisite

- A concourse environment for running pipeline. You can create concourse using Azure [template](https://github.com/Azure/azure-quickstart-templates/tree/master/concourse-ci).
- A private git repo, this repo is used to store CATs ingegration file.
- An Azure service principal. It is used by bosh director to create resources. You can learn how to create service princiapl [here](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2Fazure%2Fazure-resource-manager%2Ftoc.json&view=azure-cli-latest).

## Parameters

There are two parameters files: `credentials.yml` , `email-credential.yml` . You should fill all of them. The following is detailed description of parameters:

- In`credential.yml`: The meaning of the prameter is written as comment in the example. Here is the example:

  ```yaml
  ---
  # Credential of Azure
  TENANT_ID: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  CLIENT_ID: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
  CLIENT_SECRET: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

  # The resource group name and region of cf, this resource group will be safely delete if pipeline success.
  RESOURCE_GROUP_NAME: myResourceGroup
  AZURE_REGION: eastasia

  # SSH public key and private key of the dev box, you can create it using `ssh-keygen -t rsa -b 2048`. 
  SSH_PUBLIC_KEY: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCrCRxeXFnQFmBOjRjMactuiY5JUdrYpwJx6WhZw433hicqTbTm9SiqbyOioNM9vqvn0cuTzzIW0+715x2FgKbnFeTZnkY9dcjNuI0NkhF+Ps9X5SZrBPt1muYWs5CqW+jHQeutiCN1rJO6Hn3MyW3HjGJ/lz+x1zZQFFgsIR8S5DywNPCmzKwO3/zxE78ouqSq/QHAUZpVsxunXhtjsz/UQK/XX2J1aP6SI+fxu3rAxHh6yFuEqe0FHqIygmVZkUUc3JkclQU94lzCrrhE5+i8VZzlrOzfkJsdkaqlXOXbfVYmhnUWt49SA79hkwbszA7ucFnQoJLAKYWqzOia3Al3"
  SSH_PRIVATE_KEY: |
      -----BEGIN RSA PRIVATE KEY-----
      ...
      The private key
      ...
      -----END RSA PRIVATE KEY-----

  # Information of the cats config repo, the private key should have write access to the repo.
  CONFIG_REPO_URI: git@github.com:username/repo.git
  CONFIG_BRANCH: master
  CONFIG_PRIVATE_KEY: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    The private key with write access of your repo.
    ...
    -----END RSA PRIVATE KEY-----
  CONFIG_FILE_PATH: "cats_integration_config.json"

  # Information of the azure template repo, changes will bumped to the repo, the private key should have write access to the repo
  AZURE_TEMPLATE_REPO_URI: git@github.com:username/repo.git
  AZURE_TEMPLATE_BRANCH: bump
  AZURE_TEMPLATE_PRIVATE_KEY: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    The private key with write access of your repo.
    ...
    -----END RSA PRIVATE KEY-----

  # Git information used to push changes
  GIT_COMMIT_USERNAME: my_username
  GIT_COMMIT_EMAIL: my_email@outlook.com
  ```


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


## Usage

This pipeline is automatically triggered everyday, unless CATs fails or unpredictable error occurs, you don't need to manipulate it manually. 

To setup this pipeline, use  `fly -t your_target_name sp -c pipeline.yml -l credential.yml -l email-credential.yml -p name_of_the_pipeline`.
