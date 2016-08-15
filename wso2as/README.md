# Azure Artifacts for WSO2 Application Server
The Azure Artifacts provide the resources and instructions to deploy WSO2 Application Server on Microsoft Azure.

## How to deploy

Below steps are invloved in deploying azure artifacts

* Create a Service Principal and bind it to an application
* Create the VM image
* Copy existing VM image
* Update the parameters.json
* Run the deploy.sh

You create a service principal and bind it to an application so that the Azure login process can be automated. Then if you intend to have your own VM images instead of the VM images provided by WSO2, you can create those VM images, otherwise just igonre this step and jump to the nest step where you copy already created VM images to your Azure Storage Account. After that update the parameters.json according to your deployment and run the deploy.sh .

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

Note that home page and URI can be faked since they are not going to be verified. Upon successful execution of above cammand, appId would be returned which will be used in the next step.

#### 1.5 Create service Principal

>     azure ad sp create --applicationId <appId>

Example:

>     azure ad sp create --applicationId 246e4af7-75b5-494a-89b5-363addb9f0fa

#### 1.6 Assigning roles to Service Principal

Once you created the service principal, next step is to grant permission on it, that is assigning roles. More about this can be found [here.](https://azure.microsoft.com/en-us/documentation/articles/role-based-access-built-in-roles/)

>     azure role assignment create --spn <service-principal-name> --roleName "Owner" --subscription <subscription-id>

Example: 

>     azure role assignment create --spn "http://BOSHAzureCPI" --roleName "Owner" --subscription 87654321-1234-5678-1234-678912345678

Note that as the service principal name you can use any value under service principal names that is retuned from the previous command. In the example I have used the home page but you can use the service principal ID as well ( Ex: 6dc9e2f9-d3db-46a7-b1b3-9e369e698c37). Service principal ID also known as client ID and the password given also known as the client key or credential. 

#### 1.7 Verify the Service Principal with login command

>     azure login --username <CLIENT_ID> --password <CLIENT_SECRET> --service-principal --tenant <TENANT_ID> --environment <ENVIRONMENT>

To get the tenant_ID,

>     azure account list --json

Example: 

>     azure login --username 246e4af7-75b5-494a-89b5-363addb9f0fa --password "password" --service-principal --tenant 22222222-1234-5678-1234-678912345678 --environment AzureCloud















      
      
      
      
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
