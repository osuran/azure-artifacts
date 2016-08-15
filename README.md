# WSO2 Azure Artifacts

WSO2 Azure Artifacts enables you to run WSO2 products seamlessly on Azure. This repository contains artifacts (Service and Replication Controller definitions) to deploy WSO2 products on Azure.

## Getting started

To deploy a WSO2 product on Azure, the following overall steps have to be done.

* Capture the VM images
* Upddate the parameters.json with vmImage and other parameters
* Run deploy.sh on your local machine, there you will be connected to the Azure CLI and run Azure CLI commands to deploy. 
