# ML Workbench Template with RBAC assignment per resource
NOTE: This template includes customizations to make RBAC assignments per resource. You can find the original template [here](https://github.com/Azure/MLWorkbenchTemplate/).
<br>To run with RBAC assignment, set the following parameters in azuredeploy.parameters.json:
| Parameter                              | Description                                                                                                                                                                                                                                                                                            |
|----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| principalID                                  | ObjectID of the user or group to assign access to. Has format xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.                                                                                                                                                                                        |
| roleDefID                                    | ID of the role you want user/group assigned to. Has format xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.<br>You can get this info with the the command: `Get-AzRoleDefinition -Name <role name>`.                                                 |

If you run the template (using the instructions below) with the `-runRBAC` option each service deployed will get an RBAC assignment for the user and role defined in your parameters. If you wish to make an additional assignment, run the template again with the `-runRBAC` option and a new principalID and/or roleDefID. If you wish to run the template, but are not making new RBAC assignments, run without the `-runRBAC` option or the template will experience errors (you can not make the same assignment twice).

### Resource Tagging
By default, the resource group and all resources will be tagged with a cost center (70001036 by default). You may use the deployment flag `-costTag` to change this value.

# Machine Learning Workbench Template

<br>This template will deploy the following architecture:
<br>![Architecture Diagram](https://github.com/Azure/MLWorkbenchTemplate/blob/master/MLWorkbench%20Template%20Architecture.jpg?raw=true)

**Running the MLWorkbench Template**

**Parameters**

In the azuredeploy.parameters.json you will need to set the following: (\*
denotes required parameters)

| Parameter                              | Description                                                                                                                                                                                                                                                                                            |
|----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Prefix \*                                    | A short prefix that will be added to the name of all resources                                                                                                                                                                                                                                        |
| VirtualNetworkResourceGroupName \*           | The Resource Group where the Virtual Network exists                                                                                                                                                                                                                                                    |
| VirtualNetworkName \*                        | Name of the subnet to deploy into                                                                                                                                                                                                                                                                      |
| KvObjectID \*                                | The ObjectID of the user, group, or service principle that will have access to Key Vault                                                                                                                                                                                                               |
| vmSize                                       | By default, the template chooses Standard_D2_V2 – if this size isn’t available in the location you are deploying to, you should pass a different size here (like Standard_D1_v2) you can get available machine sizes via a powershell commandlet (Get-AzVMSize –Location \<insert location ie: eastus2\>) |
| osType                                       | Default value is ubuntu, if you provide this, allowed options are: ubuntu centOs windows – these are the different marketplace skus for DataScience vms                                                                                                                                                |
| localAdminUserSecretValue                    | Script will ask for this if not provided in parameter file (it is more secure to leave it out and provide it in PowerShell when prompted) this will be the local admin username that will be set on the VM (this value will be stored and referenced from KeyVault when the template executes)         |
| LocalAdminPasswordSecretValue                | Script will ask for this if not provided in parameter file (it is more secure to leave it out and provide it in PowerShell when prompted) this will be the local admin password that will be set on the VM (this value will be stored and referenced from KeyVault when the template executes)         |

**Considerations**

This template deploys to an existing VNET and subnet. You can set these valuesin the parameters file. Note that the subnet needs to have Service Endpoints configured for KeyVault. The template currently only provides for the VNet/Subnet to be in the same subscription as the ML environment you are creating.

You will need to have a storage account (in the subscription you are creating the ML environment in) for artifacts to be uploaded to. The template will create the storage account and container if they do not exist. Note that the storage account name needs to be globally unique.

To Login to your azure subscription in local PowerShell (not necessary in cloud shell, and assumes latest az powershell module is installed locally): `Login-AzAccount`

**Ensure you are working in the appropriate subscription**

If you have access to multiple subscriptions – this will deploy to the default subscription that the PowerShell session authenticates to. Check the currently selected subscription by using: `Get-AzContext`

This will list the current default selected subscription To change this to a different subscription try the following: 
<br/>
`$context = Get-AzSubscription –SubscriptionId <paste id of subscription to switch to>` 
<br/>
`Set-AzContext $context`

**Accept the DSVM legal terms before running:**

This template deploys the Azure Marketplace image of a Data Science Virtual Machine. To successfully deploy, you must first accept the terms of use. You can read the terms [here](https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24LINUX%253a2DDATA%253a2DSCIEN). 
You can accept the terms with the following PowerShell command: 
<br/>`Get-AzMarketplaceTerms -Publisher "microsoft-ads" -Product "linux-data-science-vm-ubuntu" -Name "linuxdsvmubuntu" | Set-AzMarketplaceTerms -Accept`

**Deploy**

All files need to exist in the directory you deploy from. The following command will deploy the template:
<br/>`.\Deploy-AzureResourceGroup.ps1 -ResourceGroupLocation "<region>” -UploadArtifacts -ResourceGroupName "<defaults to MLWorkbenchTemplate if not provided>" -StorageAccountName "<name>" -StorageContainerName "<name>"`

Other optional Parameters:
<br/>`-ValidateOnly` - will only validate that the template is valid with the provided parameters (will prevent the script from actually deploying any artifacts)
<br/>`-DSCSourceFolder` - this is used for configuration management, which is not currently used in this template
