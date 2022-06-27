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
  --name myBackendSubnet \
  --resource-group helloDjangoResourceGroup \
  --vnet-name helloDjangoVNet   \
  --address-prefix 10.21.1.0/24
az network public-ip create \
  --resource-group helloDjangoResourceGroup \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard```

