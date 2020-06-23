# azure_scripts
list of handy azure scripts

Descripton|az cli| az powershell module|
--|--|--|
Check for "Dynamic" Public IPs|```az network public-ip list --output table  --query "[?publicIpAllocationMethod=='Dynamic']" ```|```Get-AzPublicIpAddress | select-object Name,ResourceGroupName,Location,IpAddress,PublicIpAllocationMethod | Where-Object {$_.PublicIpAllocationMethod -eq "Dynamic"} | ft```|
Check for "Dynamic" Private IPs|```az network nic list --query '[].ipConfigurations[]'  --output table | egrep -i 'PrivateIpAllocationMethod | *---* | dynamic'```|```Get-AzNetworkInterface | select-object -expandproperty IpConfigurations |select-object Name,PrivateIpAddress,PrivateIpAllocationMethod | Where-Object {$_.PrivateIpAllocationMethod -eq "Dynamic"}```|
