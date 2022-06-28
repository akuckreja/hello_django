Create the resource group:
```az group create --name helloDjangoResourceGroup --location eastus```

Create network resources:
```az network vnet create \
  --name helloDjangoVNet \
  --resource-group helloDjangoResourceGroup \
  --location eastus \
  --address-prefix 10.21.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.21.0.0/24
az network vnet subnet create \
  --name helloDjangoBackEndSubnet \
  --resource-group helloDjangoResourceGroup \
  --vnet-name helloDjangoVNet   \
  --address-prefix 10.21.1.0/24
az network public-ip create \
  --resource-group helloDjangoResourceGroup \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard```

```curl https://raw.githubusercontent.com/akuckreja/hello_django/master/cloud-init.txt > cloud-init.txt```

```  az network nic create \
    --resource-group helloDjangoResourceGroup \
    --name hello_django-nic-01 \
    --vnet-name helloDjangoVNet \
    --subnet helloDjangoBackEndSubnet ```

```   az vm create \
    --image "Canonical:0001-com-ubuntu-server-focal:20_04-lts-gen2:latest" \
    --resource-group helloDjangoResourceGroup \
    --nics hello_django-nic-01 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --name hello_django-vm-01 \
    --custom-data cloud-init.txt ```

Todo: figure out how to get the python app to run at startup