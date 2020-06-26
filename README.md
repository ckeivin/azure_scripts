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
```powershell
Get-AzNetworkInterface | select-object -expandproperty IpConfigurations `
| select-object Name,PrivateIpAddress,PrivateIpAllocationMethod `
| Where-Object {$_.PrivateIpAllocationMethod -eq "Dynamic"}
```

## Storage Accounts
### List Storage Accounts with selected fields
```bash
az storage account list --query \
'[].{name:name,ResourceGroup:resourceGroup,location:location,enableHttpsTrafficOnly:enableHttpsTrafficOnly,kind:kind,accessTier:accessTier,sku_name:sku.name,sku_tier:sku.tier}' \
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
'[].{name:name,DataDisks:storageProfile.dataDisks[].managedDisk.id}' \
--output table 
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
az webapp list --query '[].{name:name,location:location,resourceGroup:resourceGroup,kind:kind,defaultHostName:defaultHostName}' \
--output table
```