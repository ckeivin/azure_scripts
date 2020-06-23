# azure_scripts
list of handy azure scripts

## Public IPs
### Check for Dynamic Public IPs
```bash
az network public-ip list --output table  --query "[?publicIpAllocationMethod=='Dynamic']"
```
```powershell
Get-AzPublicIpAddress | select-object Name,ResourceGroupName,Location,IpAddress,PublicIpAllocationMethod | Where-Object {$_.PublicIpAllocationMethod -eq "Dynamic"} | ft
```
### Check for Dynamic Private IPs
```bash
az network nic list --query '[].ipConfigurations[]'  --output table | egrep -i 'PrivateIpAllocationMethod | *---* | dynamic' 
```
```powershell
Get-AzNetworkInterface | select-object -expandproperty IpConfigurations |select-object Name,PrivateIpAddress,PrivateIpAllocationMethod | Where-Object {$_.PrivateIpAllocationMethod -eq "Dynamic"}
```

## Storage Accounts
### List Storage Accounts with selected fields
```bash
az storage account list --query '[].{name:name,ResourceGroup:resourceGroup,location:location,enableHttpsTrafficOnly:enableHttpsTrafficOnly}' --output table
```