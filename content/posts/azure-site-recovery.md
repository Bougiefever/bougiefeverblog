---
title: "Azure Site Recovery"
date: 2018-10-25T15:38:45-05:00
draft: true
---
Azure Site Recovery (ASR) is a disaster recovery service for Azure VMs. In the event of a disaster, you can failover your VMs to a different Azure region. ASR will recreate your Virtual Network, and create the new VMs using point-in-time recovery points. For some applications, it is important to failover certain VMs together as a set. For example, if you have several VMs that are each nodes for a service, they need to failover as a group. In addition, you may have a requirement that the VMs be brought back online in a certain order. ASR can handle all of that. 

### Set up a test environment
First, let's set up a couple of VMs so we can see how it works.

#### Create a resource group
1. Go to the Azure portal and click the green plus in the upper left corner to create a new item.
{{< figure src="/images/asr/create_rg_1.png" height="311" width="300" >}}
2. Type "Resource" and press enter. Select "Resource Group" to create a new resource group. Click the "Create button" in the blade that opens up.
{{< figure src="/images/asr/create_rg_2.png" height="311" width="300" >}}
3. Fill in the new resource group name, subscription, and region. Items in the resource group can be located in a different region. This is simply a logical grouping of resources. 
{{< figure src="/images/asr/create_rg_3.png" height="311" width="300" >}}

#### Create Virtual Network
We will create just one virtual network that will contain all of our assets.

1. In the resource group you just created, click the "Add" button.
{{< figure src="/images/asr/vnet_1.png" width="600" >}}
2. Define the name, address space, and subnet information. 
{{< figure src="/images/asr/vnet_2.png" width="200" >}}
    1. Name of virtual network
    2. Address space in CIDR notation
    1. Select your subscription
    1. Select the resource group you just created
    1. Select a region. All of the other assets should also be created in this region
    1. Name your subnet or accept the name "default"
    1. Select the address space for the subnet. You can accept the default.

#### Create VMs
We will create a couple of VMs to test with.
1. Click the green "Add a resource" button in the upper left corner of the Azure portal and select "Compute". Click on "See all" to get an expanded list of available VMs in the Azure Marketplace.
{{< figure src="/images/asr/vm_1.png" width="600" >}}
2. You can narrow the selection by operating system.
{{< figure src="/images/asr/vm_2.png" width="500" >}}
2. Fill out the basic information for a new VM. 
{{< figure src="/images/asr/vm_3.png" >}}

   1. Select your subscription
   2. Select your resource group
   3. Give the VM a name that will be unique in your virtual network
   1. Select the same region as your resource group
   1. Make sure the image selected is the one you want
   1. You can change the size of the VM by clicking "Change size"
   1. Select "Password" for authentication type, and fill in the username and password you want to use. Make a note of it since you won't be able to view the password later.
   1. Open up the SSH port so you can log into your VM using SSH.
{{< figure src="/images/asr/vm_4.png" >}}  
   1. Select OS disk type
   1. Add any data disks
 {{< figure src="/images/asr/vm_5.png" >}} 
   1. Select the virtual network you created earlier
   1. Select the subnet
   1. Accept the name for the public IP address
 {{< figure src="/images/asr/vm_6.png" >}} 
   1. Click "Review + create" to validate your selections
 {{< figure src="/images/asr/vm_7.png" >}} 
   1. On the final screen, make sure you see "Validation passed", and then click "Create"

After a few minutes, you should see "Your deployment is complete". Set up a second VM similar to the first one so we can test failing over a group of VMs together.
 {{< figure src="/images/asr/vm_8.png" >}} 

### Setting up Azure Site Recovery
We will set up Azure Site Recovery to test failing over our VMs. To test the VM failover, 
#### Create a Recovery Vault

