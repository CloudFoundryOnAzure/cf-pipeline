# cf-pipeline

## Introduction
This repo is used for set up common used pipeline for cloud foundry on Azure.

## Module
[bosh-setup-template](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/bosh-setup-template): This pipeline can check new version of [cf-deployment](https://github.com/cloudfoundry/cf-deployment), if any new version is found, the pipeline will use [template](https://github.com/Azure/azure-quickstart-templates/tree/master/bosh-setup) and latest manifest to deploy a cloud foundry. Then cats will run. If cats passes, changes of manifest will bump to a specified repo/branch and all allocated resources will be safely deleted.

[cf-deployment](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/cf-deployment): This is cf-deployment pipeline for Azure. More Detailed doc is coming soon...

[run-cats](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/run-cats): This pipeline is used to run cats on existing cloud foundry environment. See detailed doc [here](https://github.com/CloudFoundryOnAzure/cf-pipeline/blob/master/run-cats/README.md)

[lib](https://github.com/CloudFoundryOnAzure/cf-pipeline/tree/master/lib): This is utils used in other pipeline. You may not use it directly.
