# Azure Artifacts for WSO2 Application Server
The Azure Artifacts provide the resources and instructions to deploy WSO2 Application Server on Microsoft Azure.

## How to deploy

### 1) Capture the Virtual Machine. 
* Log into to your VM from a SSH client and execute the following command.

$sudo waagent -deprovision+user
type 'y' to continue and then exit the ssh client

* Then log into to your Azure subscription from Azue CLI in your local machine. If you have not installed Azure CLI click [here](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/):
azure login

* Make sure you are in Resource Manager mode:
azure config mode arm

* Stop the VM which you already deprovisioned by using the following command:
azure vm deallocate -g your-resource-group-name -n your-virtual-machine-name

* Generalize the VM with the following command:
azure vm generalize –g your-resource-group-name -n your-virtual-machine-name

* Now capture the image and a local file template with the following command:
azure vm capture your-resource-group-name your-virtual-machine-name your-vhd-name-prefix -t path-to-your-template-file-name.json

### 2) Update the parameters.json with the infromation of the Virtual machine that you are going to create.
      As the vmImage parameter, provide the link to the captured VM image. Follow the below path to find the VM image which is stored in the storage account where the origilan VM was resided. 
      {your storage account}>>Blobs>>System>>Microsoft.Compute>>Images>>vhds>>{VM_Image}>>UR
      
### 3) Run the deploy.sh by providing required command line arguments. 
