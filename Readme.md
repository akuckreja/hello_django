Create the resource group:
```az group create --name helloDjangoResourceGroup --location eastus```

Create network resources:
```
az network vnet create \
  --name helloDjangoVNet \
  --resource-group helloDjangoResourceGroup \
  --location eastus \
  --address-prefix 10.21.0.0/16 \
  --subnet-name helloDjangoAppGatewaySubnet \
  --subnet-prefix 10.21.0.0/24
az network vnet subnet create \
  --name helloDjangoBackEndSubnet \
  --resource-group helloDjangoResourceGroup \
  --vnet-name helloDjangoVNet   \
  --address-prefix 10.21.1.0/24
az network public-ip create \
  --resource-group helloDjangoResourceGroup \
  --name helloDjangoPublicIP \
  --allocation-method Static \
  --sku Standard
  ```

Pull down the cloud-init.txt which is used to set up the Virtual Machine on the first boot
```curl https://raw.githubusercontent.com/akuckreja/hello_django/master/cloud-init.txt > cloud-init.txt```

Create the network interface for the virtual machine we are about to create
```  az network nic create \
    --resource-group helloDjangoResourceGroup \
    --name hello_django-nic-01 \
    --vnet-name helloDjangoVNet \
    --subnet helloDjangoBackEndSubnet ```

Create the virtual machine
```   az vm create \
    --image "Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest" \
    --resource-group helloDjangoResourceGroup \
    --nics hello_django-nic-01 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --name hello_django-vm-01 \
    --custom-data cloud-init.txt ```

Create the application gateway
``` address1=$(az network nic show --name hello_django-nic-01 --resource-group helloDjangoResourceGroup | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",') ```

```az network application-gateway create --name hello_django_app_gateway --location eastus  --resource-group helloDjangoResourceGroup  --capacity 2  --sku Standard_v2  --public-ip-address helloDjangoPublicIP  --vnet-name helloDjangoVNet  --subnet helloDjangoAppGatewaySubnet --servers "$address1" --priority "1" --http-settings-port 8080 ```

Print the address for the application gateway
``` publicIp=$(az network public-ip show -g helloDjangoResourceGroup -n helloDjangoPublicIP | grep "\"ipAddress\":" | grep -oE '[^ ]+$' | tr -d '",')
echo "Address is http://$publicIp/"
 ```

Create a network interface for the second virtual machine
```
az network nic create \
    --resource-group helloDjangoResourceGroup \
    --name hello_django-nic-02 \
    --vnet-name helloDjangoVNet \
    --subnet helloDjangoBackEndSubnet
```

Create the second virtual machine
```   az vm create \
    --image "Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest" \
    --resource-group helloDjangoResourceGroup \
    --nics hello_django-nic-02 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --name hello_django-vm-02 \
    --custom-data cloud-init.txt
```

Add the IP address of the second virtual machine for the backend pool for the application gateway
``` address2=$(az network nic show --name hello_django-nic-02 --resource-group helloDjangoResourceGroup | grep "\"privateIpAddress\":" | grep -oE '[^ ]+$' | tr -d '",') 

az network application-gateway address-pool update -g helloDjangoResourceGroup --gateway-name hello_django_app_gateway -n appGatewayBackendPool --add backendAddresses ipAddress=$address2
```