# ES-storage


# Design considerations
The following is a list of design considerations when it comes to storage in AKS. Consider where storage is required in your AKS environment, and detirmine the best solution for each requirement.

* **Operating system disks**: Each virtual machine (VM) in Azure requires a disk for its operating system. Given Kubernetes nodes are ephemeral AKS defaults to using [ephemeral OS drives](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#ephemeral-os) on supported VM sizes. If required, it is possible to use regular managed disks in stead for the nodes in your AKS cluster. 
* **Application data**: Some applications need a consistent data store to store application data. If a database is required, consider exploring the managed database options in Azure such as [Azure SQL](https://azure.microsoft.com/en-us/products/azure-sql/), [Azure Database by MySQL](https://azure.microsoft.com/en-us/services/mysql/), [Azure Database for PostGres](https://azure.microsoft.com/en-us/services/postgresql/) or [CosmosDB](https://azure.microsoft.com/en-us/services/cosmos-db/).
* Storage solutions in AKS: If a managed database doesn't meet your application needs, AKS has multiple storage options available to store consistent data:
  * **Disk-based solutions**: 
  * **Files-based solutions**
  * **Blob**
  * **Other**  
