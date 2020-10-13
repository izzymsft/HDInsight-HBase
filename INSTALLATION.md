
## Installing HBase on Azure via Azure HDInsights 4.0

Our goal is to set up a HDInsight 4.0 Cluster running HBase 2.1 capable of supporting up to 400TB of data from clients sending in 200,000 requests per second. We will use Azure Resource Manager (ARM) templates for these tasks.

The ARM template contains the instructions for provisioning the resources and the parameter files would contain the values for the variables and parameters used within the ARM templates.

### Things to Consider When Setting up the Cluster

- Virtual Network Association
- Accelerated Writes
- Zookeeper Performance
- Head/Master Node Performance
- Region Node Performance
- BlockBlob Storage Account

#### Virtual Network
Associating the cluster with a virtual network simplifies how other resources on Azure can reach out to the HBase resources (Zookeeper, Region Servers etc). The benefits of doing this are highlighted in this document.

https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-plan-virtual-network-deployment


#### Accelerated Writes
We need to enable HBase accelerated writes. This feature allows the HBase Write Ahead Log to be placed on a Premium SSD disk which enables high throughput writes with lower latencies. In order to get high throughput reads as well it is highly recommended to use HBase accelerated writes along with a Premium BlockBlobStorage account.

#### Zookeeper Performance
We need to ensure that the Zookeeper nodes are capable of handling all the requests that are coming to it via the clients as well as the cluster resources. To accomplish this, I recommend setup nodes with at least 32GB of RAM and a minimum of 8 CPUs to also maximize disc speed and network bandwidths available to these ZK nodes. I would go with E4 v3 or D12 v2 and give at least 16GB of RAM to Zookeeper in the configs.


#### Head/Master Node Performance
The 2 head nodes have to be capable of running all the leader resources necessary to support the cluster. For this I recommend VMs with a minimum of 64GB RAM with at least 16 CPUs. I would go with D14 v2 instances.

#### Region/Worker Node Performance
The worker nodes have to be capable of running all the tasks necessary to support the cluster. For this I recommend VMs with a minimum of 64GB RAM with at least 16 CPUs. I would recommended going with DS14 v2 instances to provide maximum performance.

#### Block Blob Storage Account Type
In order to get high throughput reads as well it is highly recommended to use HBase accelerated writes along with a Premium BlockBlobStorage account.


### Installation Sequence

For this exercise, we will use ARM templates to ensure we get consistent results during each setup.

Before we proceed, we will first set up the pre-requisite dependencies needed to provision the cluster which are:

- Resource Group
- Azure Virtual Network
- BlockBlob Storage Account

Once these resources are available and ready for use, we can then proceed to provision the HDInsight cluster.

#### Creating the Resource Group

This resource group will house all our resources for this exercise.

```shell

az group create --name elephant --location westus2

```

#### Setting up Azure Virtual Network

We will create a VNet called dolphin with IP range 10.250.0.0/16 and then we will set up the following subnets within it.

- 10.250.0.0/20 - hdi
- 10.250.16.0/20 - bastion
- 10.250.32.0/20 - datastores

 We will use the **hdi** subnet for the cluster resources, the **bastion** subnet for our edge/bastion nodes and the **datastores** subnet for other datastores if needed.

Let's navigate to the arm-templates/virtual-network folder to deploy the virtual network

```shell

cd arm-templates/virtual-network

az deployment group create -n vnetdeployment20201013 -g elephant --template-file template.json --parameters parameters.json

```

#### Setting up the Storage Account

We will now create a BlockBlob Storage account called phoenix2020. Make sure to pick something globally unique if this name is not available.

```shell

cd arm-templates/storage-accounts

az deployment group create -n storagedeploy20201013 -g elephant --template-file template.json --parameters parameters.json

```


#### Deploying the Cluster

After setting up the dependencies we can now deploy the cluster


```shell

cd arm-templates/hdi-cluster

az deployment group create -n hbaseclusterdeploy20201013 -g elephant --template-file template.json --parameters parameters.json

```

This will allow you to log in to the AMBARI portal via the following URL using the admin username and password from the parameter file of the Cluster deployment

https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-manage-ambari

https://CLUSTERNAME.azurehdinsight.net

You can also log into one of the head nodes as follows using the SSH username and password from your parameters file:

```shell

ssh sshUserName@clusterName.azurehdinsights.net

```

https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix



