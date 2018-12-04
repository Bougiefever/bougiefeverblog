---
title: "Azure Site Recovery"
date: 2018-10-25T15:38:45-05:00
draft: false
---
Azure Site Recovery (ASR) is a disaster recovery service for Azure VMs. In the event of a disaster, you can failover your VMs to a different Azure region. ASR will recreate your Virtual Network, and create the new VMs using point-in-time recovery points. For some applications, it is important to failover certain VMs together as a set. For example, if you have several VMs that are each nodes for a service, they need to failover as a group. In addition, you may have a requirement that the VMs be brought back online in a certain order. ASR can handle all of that. 

[Set up a Test Environment](#test)

  * [Create a resource group](#create_resource_group)
  * [Create a virtual network](#Create_a_virtual_network)
  * [Create VMs](#create_vms)

[Azure Site Recovery Replication](#replication)

  * [Create a Recovery Vault](#Create_a_Recovery_Vault)
  * [Replicate VMs](#replicate_vms)

## <a name="test"></a>Set up a test environment
First, let's set up a couple of VMs so we can see how it works.

#### <a name="create_resource_group"></a>Create a resource group
1. Go to the Azure portal and click the green plus in the upper left corner to create a new item.
{{< figure src="/images/asr/create_rg_1.png" height="311" width="300" >}}
2. Type "Resource" and press enter. Select "Resource Group" to create a new resource group. Click the "Create button" in the blade that opens up.
{{< figure src="/images/asr/create_rg_2.png" height="311" width="300" >}}
3. Fill in the new resource group name, subscription, and region. Items in the resource group can be located in a different region. This is simply a logical grouping of resources. 
{{< figure src="/images/asr/create_rg_3.png" height="311" width="300" >}}

#### <a name="Create_a_virtual_network"></a>Create a virtual network
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

#### <a name="create_vms"></a>Create VMs
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

## <a name="replication"></a>Setting up Azure Site Recovery Replication
We will set up Azure Site Recovery in the paired region. Although you can use a non-paired region for your recovery region, there are some benefits to using paired regions. When Microsoft rolls out Azure updates, they don't update both regions in a pair at the same time. If an update causes an outage, only one region would be affected. If any kind of outage would affect both regional pairs, one region would be prioritized. That way at least one of the regions would be brought back online. In addition, the regions usually have high-speed connections between them, so data transfers between reginal pairs will be quicker than using a different region. All of the regional pairs are listed [here.](https://docs.microsoft.com/en-us/azure/best-practices-availability-paired-regions)

Azure Site Recovery must be in a different region than the one that contains the assets for obvious reasons. We will now create a new recovery vault in the paired region.

#### <a name="Create_a_Recovery_Vault"></a>Create a Recovery Vault
1. Click "Create a resource" and type "Recovery" to search the Azure Marketplace. Select "Backup and Site Recovery (OMS)" and click "Create".
1. Fill in the name of the recovery vault and select the subscription.
1. Click "Create new" and enter a name for your resource group in the recovery region. It's a good idea to keep this separate from the original region for organization.
1. Select the location to be the regional pair for the location of your VMs.
1. Click "Create"
{{< figure src="/images/asr/recovery_2.png" >}} 

#### <a name="replicate_vms"></a>Replicate VMs


1. On the overview blade, click +Replicate to set up replication for our first VM.
{{< figure src="/images/asr/recovery_3.png" >}} 

##### Step 1
1. Fill in the information for the source region and click "OK.
{{< figure src="/images/asr/recovery_4.png" >}} 

##### Step 2
1. In step 2, select both virtual machines for replication. Click "OK" to go to step 3.
{{< figure src="/images/asr/replication_2.png" >}}

##### Step 3
Step 3 requires some customization to set things up in the region we already created. For this example, I created a storage account in the source region, and a new virtual network in the DR region. 


##### Customize 
1. Make sure that the correct subscription is selected. 
{{< figure src="/images/asr/replication_1.png" >}}

1. Click "Customize" next to "Resource group, Network, Storage and Availability" to change the default settings.
  {{< figure src="/images/asr/replication_5.png" width="600" >}}
  1. Select the resource group created in the DR region.
  1. Allow the tool to create a new virtual network, or select one you have created yourself.
  1. Select a storage account from the source region (v1), or allow the tool to create a new one.

#### <a name="replication_group"></a>Replication Group
A replication group will set point-in-time recovery points for several VMs at the same time. If you have a workload that requires the recovery point to be coordinated between several VMs, you need to create a replication group.

1. Click "Customize next to "Replication policy" to set up a replication group. 
{{< figure src="/images/asr/replication_6.png" >}}
{{< figure src="/images/asr/replication_4.png" >}}
  1. Under "Multi-VM consistency", select "Yes".
  1. Create a new replication group.
  1. Enter a name for the new replication group.
  1. Select the machines in the group and click "OK".
1. Click "Create target resources" and wait for the resources to be created.
1. To start replicating the machines, you must click the "Enable replication" button at the bottom of the blade.

