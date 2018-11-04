---
title: "Azure Site Recovery"
date: 2018-10-25T15:38:45-05:00
draft: false
---

# What is Azure Site Recovery? 
Azure Site Recovery (ASR) is a disaster recovery service for Azure VMs. In the event of a disaster, you can failover your VMs to a different Azure region. ASR will recreate your Virtual Network, and create the new VMs using point-in-time recovery points. For some applications, it is important to failover certain VMs together as a set. For example, if you have several VMs that are each nodes for a service, they need to failover as a group. In addition, you may have a requirement that the VMs be brought back online in a certain order. ASR can handle all of that. 

## Set up a test environment
First, let's set up a couple of VMs so we can see how it works.

### Create a resource group
Go to the Azure portal and click the green plus in the upper left corner to create a new item.

{{< figure src="/images/asr/create_rg_1.png" >}}

### Create Virtual Network
### Create VMs

# Setting up Azure Site Recovery
## Create Recovery Vault

