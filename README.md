# Build a Secure Virtual Network - Azure, using the CLI

## Introduction
Design and implement a secure virtual network in Azure with virtual machines (VMs) and a storage account. Set up Network Security Groups (NSGs) for traffic control, establish secure communication through a VPN Gateway, and enable monitoring to track the network's performance and security status.

## Setting Up the CLI

## Steps Overview
1. Create a Virtual Network (VNet) with Subnets
2. Deploy VMs in Subnets
3. Set Up a Network Security Group (NSG) and Rules
4. Establish a VPN Gateway for Secure Communication
5. Create and Secure an Azure Storage Account
6. Enable Monitoring for the Network

## Step 1: Create a Virtual Network with Subnets
 - Create a virtual network (VNet) with multiple subnets for isolating workloads.

        # Create a resource group
        az group create --name SecureNetworkRG --location eastus

        # Create a virtual network
        az network vnet create \
            --name MySecureVNet \
            --resource-group SecureNetworkRG \
            --address-prefix 10.1.0.0/16 \
            --subnet-name FrontendSubnet \
            --subnet-prefix 10.1.1.0/24

        # Add a backend subnet
        az network vnet subnet create \
            --name BackendSubnet \
            --resource-group SecureNetworkRG \
            --vnet-name MySecureVNet \
            --address-prefix 10.1.2.0/24

## Step 2: Deploy VMs in Subnets
 - Deploy two virtual machines: one in the `FrontendSubnet` and another in the `BackendSubnet`.
 - The frontend VM will handle external traffic, while the backend VM will remain isolated within the VNet.

 - Deploy Frontend VM:

        az vm create \
        --resource-group SecureNetworkRG \
        --name FrontendVM \
        --vnet-name MySecureVNet \
        --subnet FrontendSubnet \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --public-ip-address "" # Make it internal by disabling the public IP

 - Deploy Backend VM:

        az vm create \
        --resource-group SecureNetworkRG \
        --name BackendVM \
        --vnet-name MySecureVNet \
        --subnet BackendSubnet \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --public-ip-address "" # Private internal VM

## Step 3: Set Up a Network Security Group (NSG) and Rules 
 - Create an NSG and add rules to control access to the frontend and backend VMs. 
 - The backend VM will only accept traffic from the frontend VM.

 - Create NSG for Frontend:

        az network nsg create \
        --resource-group SecureNetworkRG \
        --name FrontendNSG

 - Create NSG for Backend:

        az network nsg create \
        --resource-group SecureNetworkRG \
        --name BackendNSG

 - Add Rule to Allow SSH on Frontend VM:

        az network nsg rule create \
        --resource-group SecureNetworkRG \
        --nsg-name FrontendNSG \
        --name Allow-SSH \
        --priority 1000 \
        --protocol Tcp \
        --direction Inbound \
        --source-address-prefix Internet \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range 22 \
        --access Allow

 - Add Rule to Allow Backend VM Traffic Only from Frontend VM:

        az network nsg rule create \
        --resource-group SecureNetworkRG \
        --nsg-name BackendNSG \
        --name Allow-Frontend-To-Backend \
        --priority 1000 \
        --protocol Tcp \
        --direction Inbound \
        --source-address-prefix 10.1.1.0/24 \
        --source-port-range "*" \
        --destination-address-prefix "*" \
        --destination-port-range "*" \
        --access Allow

 - Associate NSGs with Subnets:

        # Associate Frontend NSG with FrontendSubnet
        az network vnet subnet update \
        --resource-group SecureNetworkRG \
        --vnet-name MySecureVNet \
        --name FrontendSubnet \
        --network-security-group FrontendNSG

        # Associate Backend NSG with BackendSubnet
        az network vnet subnet update \
        --resource-group SecureNetworkRG \
        --vnet-name MySecureVNet \
        --name BackendSubnet \
        --network-security-group BackendNSG

## Step 4: Establish a VPN Gateway for Secure Communication
 - A VPN Gateway will allow the establishment of a secure, encrypted communication between the Azure VNet and the on-premises network.

 - Create a VPN Gateway Subnet:

        az network vnet subnet create \
        --resource-group SecureNetworkRG \
        --vnet-name MySecureVNet \
        --name GatewaySubnet \
        --address-prefix 10.1.3.0/27

 - Create Public IP for VPN Gateway:

        az network public-ip create \
        --resource-group SecureNetworkRG \
        --name VPNGatewayPublicIP \
        --allocation-method Dynamic

 - Create the VPN Gateway:

        az network vnet-gateway create \
        --resource-group SecureNetworkRG \
        --name MyVPNGateway \
        --vnet MySecureVNet \
        --public-ip-address VPNGatewayPublicIP \
        --gateway-type Vpn \
        --vpn-type RouteBased \
        --sku VpnGw1 \
        --no-wait

## Step 5: Create and Secure an Azure Storage Account
 - An Azure Storage account will be used to store data securely. 
 - Access will be restricted to the VNet using private endpoints.

 - Create Storage Account:

        az storage account create \
        --name mystorageaccountsecure \
        --resource-group SecureNetworkRG \
        --location eastus \
        --sku Standard_LRS \
        --kind StorageV2

 - Add a Private EndPoint:

        az network private-endpoint create \
        --name MyPrivateEndpoint \
        --resource-group SecureNetworkRG \
        --vnet-name MySecureVNet \
        --subnet BackendSubnet \
        --private-connection-resource-id "/subscriptions/{subscription-id}/resourceGroups/SecureNetworkRG/providers/Microsoft.Storage/storageAccounts/mystorageaccountsecure" \
        --connection-name storageprivateendpoint

## Step 6: Enable Monitoring for the Network
 - Enable monitoring for your VMs and virtual network to gain insight into traffic patterns and performance.

  - Enable Network Watcher:

        az network watcher configure \
        --locations eastus \
        --resource-group SecureNetworkRG \
        --enabled true

 - Monitor Traffic with NSG Flow Logs:

        az network watcher flow-log configure \
        --resource-group SecureNetworkRG \
        --nsg-name FrontendNSG \
        --enabled true \
        --storage-account mystorageaccountsecure \
        --retention 30

 - Enable VM Insights for Monitoring:

        az vm monitor metrics tail \
        --resource-group SecureNetworkRG \
        --name FrontendVM

## Final Step: Release Resources (Optional)

End.