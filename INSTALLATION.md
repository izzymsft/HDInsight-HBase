
## Installing HBase on Azure via Azure HDInsights 4.0

Our goal is to set up a HDInsight 4.0 Cluster running HBase 2.1

### Things to Consider When Setting up the Cluster

- Virtual Network Association
- Accelerated Writes
- Zookeeper Performance
- Head/Master Node Performance
- Region Node Performance
- BlockBlob Storage Account

#### Virtual Network
Associating the cluster with a virtual network simplifies how other resources on Azure can reach out to the HBase resources (Zookeeper, Region Servers etc)

#### Accelerated Writes
We need to enable HBase accelerated writes. This feature allows the HBase Write Ahead Log to be placed on a Premium SSD disk which enables high throughput writes with lower latencies. In order to get high throughput reads as well it is highly recommended to use HBase accelerated writes along with a Premium BlockBlobStorage account.

#### Zookeeper Performance
We need to ensure that the Zookeeper nodes are capable of handling all the requests that are coming to it via the clients as well as the cluster resources. To accomplish this, I recommend setup nodes with at least 32GB of RAM and a minimum of 8 CPUs to also maximize disc speed and network bandwidths available to these ZK nodes.


#### Head/Master Node Performance
The 2 head nodes have to be capable of running all the leader resources necessary to support the cluster. For this I recommend VMs with a minimum of 64GB RAM with at least 16 CPUs.

#### Region Node Performance
The worker nodes have to be capable of running all the tasks necessary to support the cluster. For this I recommend VMs with a minimum of 64GB RAM with at least 16 CPUs.

#### Block Blob Storage Account Type
In order to get high throughput reads as well it is highly recommended to use HBase accelerated writes along with a Premium BlockBlobStorage account.


### Installation Sequence

For this exercise, we will use ARM templates to ensure we get consistent results during each setup.

Before we proceed, we will first set up the pre-requisite dependencies needed to provision the cluster which are:

- Azure Virtual Network
- BlockBlob Storage Account

Once these resources are available and ready for use, we can then proceed to provision the HDInsight cluster