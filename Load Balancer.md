![Architecture diagram as explained in the text.](https://learn.microsoft.com/en-us/training/wwl-azure/configure-azure-load-balancer/media/lab-06.png)

## Deploy Virtual machines:
Create a security group:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/8a8a7177-bda3-4a3f-a533-181e5ff4d173)

In Powershell select the advanced settings and create storage and create a file share:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/feee6309-06be-4de2-87f5-24686c671878)
Upload the - az104-06-vms-loop-parameters.json and az104-06-vms-loop-template.json
The json can be downloaded from this github repo:
https://github.com/MicrosoftLearning/AZ-104-MicrosoftAzureAdministrator/tree/master/Allfiles/Interactive%20Lab%20Simulation%20Files/06
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/a6435a87-dd5e-4f9c-96a2-15d483e718e2)
Change the desired password:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/d6b17514-41bf-4c86-afe2-defc9053de8c)
According to the documentation the below script runs the deployment of the VMs:
```sh
New-AzResourceGroupDeployment -Name "az104-04-vms" -ResourceGroupName "DNS-configuration" -TemplateParameterFile "az104-06-vms-loop-parameters.json" -TemplateFile "az104-06-vms-loop-template.json"
```
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/fc37a1cb-3694-42e9-84ea-68bd59ae3f49)

The command shown below will install the network watcher extension on the Azure Virtual machines.
```sh
$location =(Get-AzResourceGroup -ResourceGroupName "DNS-configuration").location
$vmNames = (Get-AzVM -ResourceGroupName "DNS-configuration").Name
foreach ($vmName in $vmNames){
Set-AzVMExtension `
-ResourceGroupName "DNS-configuration" `
-Location $location `
-VMName $vmName `
-Name 'networkWatcherAgent' `
-Publisher 'Microsoft.Azure.NetworkWatcher' `
-Type 'NetworkWatcherAgentWindows' `
-TypeHandlerVersion '1.4' 
}
```
Network watcher is installed:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/07139b4f-5302-4300-a025-38e160f997c6)

## Configure hub and spoke network topology:

Select the vnet which might act like a hub and enter into peering option. Add the peering name for the which refers to which spoke the hub is connecting to, in our case it is vnet1 to vnet2. 
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/05bbfa86-937a-49e0-8c65-80941d97a78d)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/ccc494e9-db62-4a4d-8191-99030f03b85b)

Add the peering name for the which refers to which spoke the hub is connecting to, in our case it is vnet1 to vnet3. 
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/dd1f1019-6e52-4a80-80be-06bbfda45705)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/4b394bc7-6b4a-4e9c-a7f0-88fc5bed72ce)


## Test transitivity of virtual network peering
Search for network watcher and select the ‘Connection troubleshoot’ under Network diagnostics tools tab.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/612a6b1d-48ec-439e-81fb-05de20fc80ed)
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/6e3cd614-ec7b-4c38-a43d-3580eb4c80c4)

After entering the parameters click on ‘Run diagnostic test’, below are the results of the test.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/45f193c0-df12-41d1-a77d-7097e90ce01b)

Similarly do this for the second spoke network.

Looks like one of the VMs is unreachable.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/dd5a8263-74ac-4169-9e93-f941ab52b22e)



## Configure routing in the hub and spoke topology

Get into the first virtual machine and click on the Network settings under the networking tab and click on the network interface card.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/f2b7d7ce-35b2-4d8f-a566-8db4b38030a0)
Inside the NIC select the IP configuration and enable the IP forwarding.
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/1917f970-05f8-4a23-98ab-0a8ab672c611)

Go back to the Virtual machine and under the operations tab select the Run command
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/bd5d8960-5f3d-4db8-9ea5-b1e43d3a14fc)

`Install-WindowsFeature RemoteAccess -IncludeManagementTools`
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/05e26b14-4588-4027-8e2e-80b7b255e746)

```sh
Install-WindowsFeature -Name Routing -IncludeManagementTools -IncludeAllSubFeature

Install-WindowsFeature -Name "RSAT-RemoteAccess-Powershell"

Install-RemoteAccess -VpnType RoutingOnly

Get-NetAdapter | Set-NetIPInterface -Forwarding Enabled
```

![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/fd772443-9669-4136-8c7a-19266f2c79e3)

Now create the Route tables:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/b4394197-b33d-47fb-844c-8b53f541ccd6)

After the route table is create get into the route table and add a route. We are trying to establish connect between spoke 1 vnet and vnet spoke 2. 
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/d1309946-75db-43dd-9b9b-e2307f46caf2)

Now create a subnet and associate that to the 1st spoke subnet:
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/01734276-fffd-4ab0-bf5d-ad1bb60ef614)

Similarly create route table to establish connect between spoke 2 vnet and vnet spoke 3. 
![image](https://github.com/karthi770/Jira_GitHub_intergration_Python/assets/102706119/0884dda9-a910-4e96-a95e-7f880ea20fe2)
Create the subnet association.

Now with the network watcher test the connection between the 2 networks.
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/692a0876-bed0-4e11-94fe-7873f412c71c)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/e9e91d89-3c7f-4e1e-b034-1e259d96be8e)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/ca8fd752-12be-48f2-8d88-070071b03cf6)

## Implement Azure Load Balancer
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/e84a99b0-3a04-40bf-bd80-269d87d5f4fc)

![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/c8a73b1b-a7ca-460c-859c-3abbd427e336)

In the Public IP address space click on Create New and give the details. Enter the name and make sure the zone is selected has ‘No zone’.
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/360ee405-8c88-4d25-ae34-7d04eac798eb)

Now add a backend pool, select the hub vnet and in the IP configuration tab select the IP configuration of VMs in the Hub.
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/2c513e66-dad2-488c-88ef-8641eb816e9f)![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/05f6f3b4-d9bb-4ec9-8f4b-8d49b8c14f2c)

Now add a load balancing rule:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/d625cc8d-980e-47c9-8628-dd94fe549340)
Add a health probe with the following configurations:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/20648e94-5291-4748-82fe-179785ebf81e)
The default settings shall remain the same:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/b2c779ed-cf89-47db-9607-98d9a221be8f)

Once the load balancer is created go to Frontend IP configuration:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/273b2103-1402-4fd8-b577-260c8717eacc)
Copy the public IP and visit the page: If we refresh the page multiple times we shall get the message from the other VM.
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/b219757b-574f-45f0-8c39-8a123bd9f508)

## Implement Azure Application Gateway:
Create a Subnet that shall be accessed by the Application Gateway, the subnet shall be created in the Hub virtual network:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/c6d910fe-dde0-4b35-bdb6-d9ee8693b40f)

![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/0ed49f99-0821-43de-8eff-f0791c95e02f)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/fe963e97-dcd3-425d-94c3-0b4655ea6c83)

![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/4cbfd7c3-145e-46a2-a63a-eb31c112ae0d)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/12b5a8f1-baad-4916-93aa-155053646386)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/f61bd6d4-8303-4514-9453-79f780380ee0)

Once the listeners are over click on the Backend targets and add the new backend settings:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/a57c9951-321e-47db-b589-294e1f099f6e)
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/1673f395-0f69-47d8-bab0-28bc6546b0ac)

Once the application gateway is created copy the Frontend IP address:
![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/fbc25258-0978-4e14-8173-a412d500908c)

![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/e7e9bcd1-d311-49e8-867b-e6036288053a)

![image](https://github.com/karthi770/Azure_VNet_NSG_Bastion/assets/102706119/04e542f9-6bf2-4996-9dbc-6b0483d8aa9d)

The VM changes very time we refresh the page.
