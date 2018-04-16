# run-cats

## Introduction
This pipeline is used to run cats using existing cloud foundry environment. You should have credential to log in the tested cloud foundry. Also, a private repo is need to store integration configs, it is highly recommended to create a new empty private repo. 

## Structure
This pipeline consists of two job. The first job reads cloud foundry credentials and creates integration config and uploads it to your provided repo. The second job takes the repo as input and runs official task [run-cats](https://github.com/cloudfoundry/cf-deployment-concourse-tasks/tree/master/run-cats).

## Parameters
You should change parameters in `credential.yml` to your own:
```
INTEGRATION_CONFIG_REPO_URI: <YOUR_PRIVATE_REPO_SSH_ADDRESS>
REPO_KEY: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    The private key with write access of your repo.
    ...
    -----END RSA PRIVATE KEY-----
API_ENDPOINT: api.<SYSTEM_DOMAIN>
APPS_DOMAIN: <SYSTEM_DOMAIN>
ADMIN_USER: <ADMIN_USER>
ADMIN_PASSWORD: <CF_ADMIN_PASSWORD>
GIT_COMMIT_USERNAME: <USERNAME_USED_TO_PUSH_CHANGES>
GIT_COMMIT_EMAIL: <EMAIL_USED_TO_PUSH_CHANGES>
```

## Usage
You should already have a [concourse environment](https://github.com/Azure/azure-quickstart-templates/tree/master/concourse-ci). Download this folder and fill in `credential.yml`, then run `fly -t your_target_name sp -c pipeline.yml -l credential.yml -p name_of_the_pipeline` to create the pipeline.
