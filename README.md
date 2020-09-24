# azure_scripts
list of handy azure scripts

## Public IPs
### Check for Dynamic Public IPs
```bash
az network public-ip list --output table  \
--query "[?publicIpAllocationMethod=='Dynamic']"
```
```powershell
Get-AzPublicIpAddress `
| select-object Name,ResourceGroupName,Location,IpAddress,PublicIpAllocationMethod `
| Where-Object {$_.PublicIpAllocationMethod -eq "Dynamic"} | ft
```
### Check for Dynamic Private IPs
```bash
az network nic list --query '[].ipConfigurations[]'  --output table \
| egrep -i 'PrivateIpAllocationMethod | *---* | dynamic' 
```

```bash
az network nic list --query '[].{name:name,resourceGroup:resourceGroup,location:location,privateIpAddress:ipConfigurations[].privateIpAddress}' \
--output table

```
```powershell
Get-AzNetworkInterface | select-object -expandproperty IpConfigurations `
| select-object Name,PrivateIpAddress,PrivateIpAllocationMethod `
| Where-Object {$_.PrivateIpAllocationMethod -eq "Dynamic"}
```

## Storage Accounts
### List Storage Accounts with selected fields
```bash
az storage account list --query \
'[].{name:name,ResourceGroup:resourceGroup,location:location,enableHttpsTrafficOnly:enableHttpsTrafficOnly,kind:kind,accessTier:accessTier,sku_name:sku.name,sku_tier:sku.tier,VnetRules:networkRuleSet.defaultAction}' \
--output table
```

## Vitual Machines

```bash
az vm list --query \
'[].{name:name,vmType:hardwareProfile.vmSize,location:location,resourceGroup:resourceGroup,availabilitySet:availabilitySet.id,zones:zones,virtualMachineScaleSet:virtualMachineScaleSet,osDiskType:storageProfile.osDisk.osType,offer:storageProfile.imageReference.offer,sku:storageProfile.imageReference.sku}' \
--output table 
```
### List VMs and their respective OS and data disks
```bash
az vm list --query \
'[].{name:name,osDisk:storageProfile.osDisk.managedDisk.id,DataDisks:storageProfile.dataDisks[*].managedDisk.id}' \
--output table 


az vm list --query \
'[].{name:name,DataDisks:storageProfile.dataDisks[*].managedDisk.id}' \
--output tsv
```

```bash
az vm list --query \
'[].[name,storageProfile.osDisk.managedDisk.id,storageProfile.dataDisks[*].managedDisk.id]' \
--output table 
``` 

## Key Vaults

```bash

keyvault_list=$(az keyvault list --query '[].name' --output tsv)

for i in $keyvault_list
do 
az keyvault show --name $i --query '{name:name,resourceGroup:resourceGroup,location:location,sku:sku.name,enableSoftDelete:properties.enableSoftDelete,softDeleteRetentionInDays:softDeleteRetentionInDays}' \
--output table
done
```

## app service plans
```bash
az appservice plan list --query '[].{name:name,resourceGroup:resourceGroup,location:location,kind:kind,sku_name:sku.name,sku_tier:sku.tier,numberOfSites:numberOfSites,status:status}' --output table
```

### web apps
```bash
az webapp list --query '[].{name:name,location:location,resourceGroup:resourceGroup,kind:kind,appServicePlanId:appServicePlanId,defaultHostName:defaultHostName}' \
--output table
```

## function app
```bash
az functionapp list --query  '[].{name:name,location:location,resourceGroup:resourceGroup,kind:kind,appServicePlanId:appServicePlanId,defaultHostName:defaultHostName}' \
--output table
```

## sql servers (PaaS)
```bash
az sql server list --query '[].{name:name,resourceGroup:resourceGroup,location:location,kind:kind,administratorLogin:administratorLogin,fullyQualifiedDomainName:fullyQualifiedDomainName}' \
--output table
```

## sql vms 
```bash
az sql vm list --query '[].{}'
```

## backup vault 


```bash
az backup item list --resource-group rg-isams-hosting-vault -v iSAMS-Hosting-Vault --query '[].[name,properties.backupEngineName,properties.backupManagementType]'

az backup item list --resource-group RG-RecoveryVault -v iSAMS-Recovery-Vault --query '[].[name,properties.backupEngineName,properties.backupManagementType]'
```
	
## network security groups
```bash
az network nsg list --query '[].{name:name,resourceGroup:resourceGroup,location:location,subnets:subnets,networkInterfaces:networkInterfaces[].id}' \
--output table

az network nsg list --query '[].{name:name,resourceGroup:resourceGroup,location:location,subnets:subnets[].id}' \
--output table


az network nsg list --query '[].{name:name,securityRules:securityRules[]}' > new_nsg.json

```

### generate nsg rules for each nsg 
```bash
nsg_id=$(az network nsg list --query '[].id' --output tsv)

for i in $nsg_id
do 
    name=$(echo $i |awk -F/ '{print $NF}')
    #az network nsg show --ids $i > nsg2/${name}-nsg.json
    # az network nsg show --ids $i --query '{name:name,securityRules:securityRules[*].name,securityRules_priority:securityRules[*].priority}' > nsg2/${name}-nsg.json

    az network nsg show --ids $i --query '{securityRules:securityRules[*]}' > ${name}-nsg.json
    sed -i "s/description/nsgName\"\:\"$name\"\,\"description/g" ${name}-nsg.json

done
```
```bash
ls |grep -noir priority | awk -F: '{print $2}
```
```bash
az network nsg show --ids $i --query '{name:name,securityRules:securityRules[*]}' \
--output tsv
```
### jq 
```bash
jq -r '.securityRules'
```

## create service principal 
```bash
# a name for our azure ad app
appName="kvtest-service-principal-1"
# create an Azure AD app
az ad app create \
    --display-name $appName \
    --homepage "http://localhost/$appName" \
    --identifier-uris http://localhost/$appName

# get the app id
appId=$(az ad app list --display-name $appName --query [].appId -o tsv)


# create a service principal
az ad sp create-for-rbac --name $appId --password $spPassword 
```

# resource groups
```bash
az resource list --output table
```
## service bus
```bash 
az servicebus  namespace list --output tableaz
```
## snapshots
```bash
az snapshot list --output table
```
## cosmosdb
```bash
az cosmosdb list --output table
```
## CDN
```bash 
az cdn profile list --query '[].{name:name,resourceGroup:resourceGroup,location:location,sku:sku.name}' --output table 
```
## app insights

## tag
```bash 
az tag list --query '[].{tagName:tagName,tagValue:values[].tagValue,tagCount:values[].count.value}' --output table
```

## search for vm images
```bash
az vm image list --publisher "debian" --all --output table 
```