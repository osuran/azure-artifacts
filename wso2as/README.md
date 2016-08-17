# Azure Artifacts for WSO2 Application Server
The Azure Artifacts provide the resources and instructions to deploy WSO2 Application Server on Microsoft Azure.

## How to deploy

Below steps are involved in deploying azure artifacts

* Create a Service Principal and bind it to an application
* Capture the VM image
* Copy existing VM image
* Update the parameters.json
* Run the deploy.sh

You create a service principal and bind it to an application so that the Azure login process can be automated. Then if you intend to have your own VM images instead of the VM images provided by a third party, you can capture those VM images, otherwise just ignore this step and jump to the next step where you copy already created VM images to your Azure Storage Account. After that update the parameters.json according to your deployment and run the deploy.sh .

### 1) Create a Service Principal and bind it to an application 

#### 1.1 Install Azure CLI if you have not done it yet. You can find help for that [here.](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)

#### 1.2 Login to your subscription
   >  azure login
   
#### 1.3 Set mode to Azure Resource Manager 
>     azure config mode arm

#### 1.4 Create Azure Active directory Application

>     azure ad app create --name <name> --password <password> --home-page <home-page> --identifier-uris <identifier-uris>

Example :

>     azure ad app create --name "sample application for wso2" --password "password" --home-page "http://wso2.org" --identifier-uris "http://wso2.org"

Note that home page and URI can be faked since they are not going to be verified. Upon successful execution of above command, appId would be returned which will be used in the next step.

#### 1.5 Create service Principal

>     azure ad sp create --applicationId <appId>

Example:

>     azure ad sp create --applicationId 246e4af7-75b5-494a-89b5-363addb9f0fa

#### 1.6 Assigning roles to Service Principal

Once you created the service principal, next step is to grant permission on it, that is assigning roles. More about role types can be found [here.](https://azure.microsoft.com/en-us/documentation/articles/role-based-access-built-in-roles/)

>     azure role assignment create --spn <service-principal-name> --roleName "Owner" --subscription <subscription-id>

Example: 

>     azure role assignment create --spn "http://BOSHAzureCPI" --roleName "Owner" --subscription 87654321-1234-5678-1234-678912345678

Note that as the service principal name you can use any value under service principal names that is returned from the previous command. In the example I have used the home page but you can use the service principal ID as well ( Ex: 6dc9e2f9-d3db-46a7-b1b3-9e369e698c37). Service principal ID also known as client ID and the password given also known as the client key or credential. 

#### 1.7 Verify the Service Principal with login command

>     azure login --username <CLIENT_ID> --password <CLIENT_SECRET> --service-principal --tenant <TENANT_ID> --environment <ENVIRONMENT>

To get the tenant_ID,

>     azure account list --json

Example: 

>     azure login --username 246e4af7-75b5-494a-89b5-363addb9f0fa --password "password" --service-principal --tenant 22222222-1234-5678-1234-678912345678 --environment AzureCloud



### 2) Capture the VM image. 

If you intend to use already captured VM images, then kindly ignore this step.

First of all you have to configure your VM with WSO2 Application Server. Following are few things to keep in mind.

* '/home/*' will be completely erased so that do not keep any file or directory in it.
* In the WSO2 documentation, you will be asked to update the .bashrc with java home. Do not update /home/<user>/.bashrc , instead update /etc/bash.bashrc with the java home.

#### 2.1 Log into to your VM from a SSH client and execute the following command.

>  sudo waagent -deprovision+user

type 'y' to continue and then exit the ssh client

#### 2.2  Then log into to your Azure subscription from Azure CLI (ignore this step if you have already logged in from the step 1.7)

>     azure login

#### 2.3 Make sure you are in Resource Manager mode (ignore this step if you are already in the arm mode)

>     azure config mode arm

#### 2.4  Stop the VM which you already deprovisioned by using the following command

>      azure vm deallocate -g  <your-resource-group-name> -n  <your-virtual-machine-name>

#### 2.5 Generalize the VM with the following command

>     azure vm generalize –g  <your-resource-group-name>  -n  <your-virtual-machine-name>

#### 2.6 Now capture the image and a local file template with the following command

>      azure vm capture <your-resource-group-name> <your-virtual-machine-name> <your-vhd-name-prefix> -t <path-to-your-template-file-name.json>

Captured VM image is stored in the same azure storage account where the original VM resided. {your storage account}>>Blobs>>System>>Microsoft.Compute>>Images>>vhds>>{VM_Image}>>URL

### 3) Copy Existing VM images

There are few ways to copy VM images within or across Azure storage accounts. 

#### 3.1 Azcopy is one of them, you can download Azcopy from [here](https://azure.microsoft.com/en-us/documentation/articles/storage-use-azcopy/)

>     AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /Pattern:abc.txt

Note that Azcopy is available only for Windows.

#### 3.2 Next method is using Azure CLI.

>     azure storage blob copy start '<https://<sourceStorageAccount>.blob.core.windows.net/mycontainer2/myBlockBlob2>' <destinationAccountContainer>

This is the prefered method since previous method needs the source and destination storage account keys. In this method, the source URL provided for the copy operation must either be publicly accessible, or include a SAS (shared access signature) token.



### 4) Update the parameters.json with the information of the Virtual machine that you are going to create.


Some sample parameter values are provided in the parameters.json file. 
As the vm_image parameter, provide the link to the captured VM image. Follow the below path to find the VM image which is stored in the storage account where the original VM was resided. 

{your storage account}>>Blobs>>System>>Microsoft.Compute>>Images>>vhds>>{VM_Image}>>URL

Sample vm_image parameter value would look like this: 

https://yourstorageaccount.blob.core.windows.net/system/Microsoft.Compute/Images/vhds/testvm1image-osDisk.a2652bad-ba56-4c0a-bd22-a651ccf8cfc8.vhd
      
### 5) Run the deploy.sh  

Subscription ID, Resource group and the deployment names must be provided either as command line arguments or when prompted, If you did not provide them as command line arguments you will be prompted to enter them later. Deployment name can be anything as it is only for the sake of naming the deployment. If the entered resource group is new, then you have to provide the location of it as well.

And then you will be asked to enter Service Principal/ Client Id, password, and tenant Id. There you can enter the values you extracted in the first section of this article. 
