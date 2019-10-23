##   Configure and Execute Terraform to provision Azure Kubernetes Cluster Resources
		
		
   # Prerequisites
		
    * Azure Cloud Account proviprovisioned  with  admin role  in order to setup 
    * Azure CLI Configured
		   
# Create an Azure service principal with Azure CLI
	
Automated tools that use Azure services should always have restricted permissions. Instead of having applications sign in as a fully privileged user, Azure offers service principals.
An Azure service principal is an identity created for use with applications, hosted services, and automated tools to access Azure resources. This access is restricted by the roles assigned to the service principal, giving you control over which resources can be accessed and at which level. For security reasons, it's always recommended to use service principals with automated tools rather than allowing them to log in with a user identity.


''''az ad sp create-for-rbac --name ServicePrincipalName''''

 * Important
The output for a service principal with password authentication includes the password key. Make sure you copy this value - it can't be retrieved. If you forget the password, reset the service principal credentials.
The appId and tenant keys appear in the output of az ad sp create-for-rbac and are used in service principal authentication. Record their values, but they can be retrieved at any point with az ad sp list.

''''az ad sp list --show-mine --query "[].{id:appId, tenant:appOwnerTenantId}"''''

Important
az ad sp list or az ad sp show get the user and tenant, but not any authentication secrets or the authentication method. Secrets for certificates in Key Vault can be retrieved with az keyvault secret show, but no other secrets are stored by default. If you forget an authentication method or secret, reset the service principal credentials.
Manage service principal roles

```az role assignment create --assignee APP_ID --role Reader```
```az role assignment delete --assignee APP_ID --role Contributor```
 Note
If your account doesn't have permission to assign a role, you see an error message that your account "does not have authorization to perform action 'Microsoft.Authorization/roleAssignments/write'." Contact your Azure Active Directory admin to manage roles.
Adding a role doesn't restrict previously assigned permissions. When restricting a service principal's permissions, the Contributor role should be removed.
The changes can be verified by listing the assigned roles:

 ```az role assignment list --assignee APP_ID

* Sign in using a service principal

Test the new service principal's credentials and permissions by signing in. To sign in with a service prinicpal, you need the appId, tenant, and credentials.
To sign in with a service principal using a password:

 ```az login --service-principal --username APP_ID --password PASSWORD --tenant TENANT_ID ```

To sign in with a certificate, it must be available locally as a PEM or DER file, in ASCII format:

az login --service-principal --username APP_ID --tenant TENANT_ID --password /path/to/cert

Reset credentials

If you forget the credentials for a service principal, use az ad sp credential reset. The reset command takes the same arguments as az ad sp create-for-rbac.

 ```az ad sp credential reset --name APP_ID ```



* Setup Azure AD service principal  in order to terraform cli to assume role and create Kubernetes cluster on 
		

 ```az ad sp create-for-rbac --name ServicePrincipalName ```

# Install Terraform

To install Terraform, download the package for your operating system into a separate installation directory. The download contains a single executable file, for which you must also define a global path. For instructions on setting the path on Linux and Mac, see this web page . For instructions on setting the path on Windows, see this web page .
Check the configuration of your path using the command terraform. A list of available Terraform options appears, as in the following output example:
console

azureuser@Azure:~$ terraform
Usage: terraform [--version] [--help] <command> [args]

# Configure Terraform access to Azure

Create an Azure AD service principal to allow Terraform to provision resources in Azure. The service principal allows your Terraform scripts to provision resources in your Azure subscription.
If you have multiple Azure subscriptions, first query your account with az account list for a list of Subscription ID and Subscriber ID values:
Azure CLI



 ```az account list --query "[].{name:name, subscriptionId:id, tenantId:tenantId}" ```
To use a selected subscription, set the subscription for this session with az account set . Set the environment variable to contain the value of the field returned from the subscription you want to use: SUBSCRIPTION_IDid
Azure CLI


 ```az account set --subscription="${SUBSCRIPTION_ID}" ```
You can now create a service master for use with Terraform. Use az ad sp create-for-rbac and set the scope to your subscription as follows:
Azure CLI


az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"
Your values appId, password, sp_nameand tenantare returned. Note the values and . appIdpassword
Configure Terraform environment variables

To configure Terraform to use your Azure AD service principal, set the following environment variables, which are then used by Azure Terraform modules . You can also set the environment if you are working on an Azure cloud other than Azure Public.
		ARM_SUBSCRIPTION_ID
		ARM_CLIENT_ID
		ARM_CLIENT_SECRET
		ARM_TENANT_ID
		ARM_ENVIRONMENT
You can use the following shell script example to set these variables:

<
#!/bin/sh
echo "Setting environment variables for Terraform"
export ARM_SUBSCRIPTION_ID=your_subscription_id
export ARM_CLIENT_ID=your_appId
export ARM_CLIENT_SECRET=your_password
export ARM_TENANT_ID=your_tenant_id

# Not needed for public, required for usgovernment, german, china
export ARM_ENVIRONMENT=public
Run a sample script>

Create a file test.tf in an empty directory and paste the following script.

<
provider "azurerm" {
}
resource "azurerm_resource_group" "rg" {
        name = "testResourceGroup"
        location = "westus"
}
Save the file and then initiate the Terraform deployment. This step downloads the Azure modules needed to create an Azure resource group.
bash>

 ``` ```terraform init ```

The result looks like the following example:

```terraform plan -o outputfile```


  provider.azurerm: version = "~> 0.3"

Terraform has been successfully initialized!
You can preview the actions to be performed by the Terraform script with terraform plan. 
When you are ready to create the resource group, apply your Terraform plan as follows:

```terraform apply```

The result looks like the following example:


An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.rg
      id:       <computed>
      location: "westus"
      name:     "testResourceGroup"
      tags.%:   <computed>

azurerm_resource_group.rg: Creating...
  location: "" => "westus"
  name:     "" => "testResourceGroup"
  tags.%:   "" => "<computed>"
azurerm_resource_group.rg: Creation complete after 1s
---

## Setup Kubernetese Cluster Environment

Please download the code https://github.com/avi202020/terraform-provider-azurerm.git

cd terraform-provider-azurerm/examples/kubernetes/basic
 
 **Please make sure environment parameters are setup execute and then exit
		ARM_SUBSCRIPTION_ID
		ARM_CLIENT_ID
		ARM_CLIENT_SECRET
		ARM_TENANT_ID
		ARM_ENVIRONMENT**

 ```execute terraform init```
 
 execute plan outfile
 
 ```terraform plan outfile```
 
Once plan is validated,execute terraform apply

  ```Terraform apply outfile```


* Connect to kubernetes Cluster

	az aks get-credentials --resource-group <> name <Cluster Name>

```kubectl get nodes```
