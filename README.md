# About Uptec

At [Uptec](https://uptec.io) we do **Cloud Differently**. One of the things we focus on is to decrease your deployment Timelines With IaC. We help you to get repeatable consist infrastructure deployments in the cloud and actively give back to the community. This is why we wanted to release this extension.

# Subnet Calculator

We often find ourselves trying to figure out which subnets are in use, which one are free or trying to calculate if the specified subnet mask would fit inside our existing network. This extension provide tasks that will let you calculate free subnets within a given network address space.

For example if you have a network with a 10.0.0.0./16 address space and you already have 2 subnets with the following subnet masks:

- SubnetA: 10.0.0.0./24
- SubnetB: 10.0.1.0/24

Now, if I need a third subnet (SubnetC), also with a /24 mask, the returned value by the task will be 10.0.2.0/24. If at a later stage I decide to delete subnetB and decide to create a 4th subnet (SubnetD) the task will calculate if the provide subnet mask fits in between subnet A and C. This is done to optimise the available space within your network.

## Prerequisites

- At this stage only Windows build agents are supported.
- Install the SubnetCalculator from the marketplace into your organisation.
- For Azure a service connection to Azure is needed. This service connection needs to have read permission on the Vnet and Virtual Network Contributor permission if you want to create subnets as well.

## New Subnet Finder

Add the "New Subnet Finder" task to your pipeline to calculate the next available free subnet. You need to provide 4 inputs for the task:

1. The existing subnets with subnet mast. The subnet needs to specified in the following format: x.x.x.x/x and comma delimeted. If you are using Azure or AWS you could grab all the existing subnets in a previous task and passed it on here.
2. The new subnet size. In this example we want to see if we can create a new /29 subnet
3. The existing Network scope. In this example we have an existing 10.0.0.0/16 network
4. The name of the output. The calculated subnet will be stored inside this output to be used in a subsequent task.

![Subnet Calculator](https://github.com/uptecio/New-Subnet-Finder/blob/main/images/NewSubnetCalculatorTask.PNG)

```yml
pool:
  vmImage: windows-latest

- task: NewSubnetFinder@0
  inputs:
    Subnets: '10.0.1.0/28,10.0.2.0/28,10.0.3.0/28'
    NewSubnetSize: '29'
    VnetScope: '10.0.0.0/16'
    OutputVariableName: 'newsubnetoutput'

- script: |
    echo $(newsubnetoutput)
  displayName: 'Run a multi-line script'
```

## New Azure Subnet Finder

This task is similar to the previous one except that we can create the subnet if required.

![Azure Subnet Calculator](https://github.com/uptecio/New-Subnet-Finder/blob/main/images/NewAzureSUbnetCalculator.png)

```yml
pool:
  vmImage: windows-latest
  
- task: NewAzureSubnetFinder@0
  inputs:
    ConnectedServiceNameARM: 'SC-TEST'
    ResourceGroupName: 'testsubnetcalculator'
    VirtualNetworkName: 'vnet'
    VirtualNetworkAddressSpace: '10.0.0.0/16'
    NewSubnetSize: '28'
    OutputVariableName: 'testazure'
```

# Pipeline Examples
