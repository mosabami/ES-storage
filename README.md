# ES-storage


# Design considerations
The following is a list of design considerations when it comes to storage in AKS. Consider where storage is required in your AKS environment, and detirmine the best solution for each requirement.

* **Operating system disks**: Each virtual machine (VM) in Azure requires a disk for its operating system. Given Kubernetes nodes are ephemeral AKS defaults to using [ephemeral OS drives](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#ephemeral-os) on supported VM sizes. If required, it is possible to use regular managed disks in stead for the nodes in your AKS cluster. 
* **Application data**: Some applications need a consistent data store to store application data. If a database is required, consider exploring the managed database options in Azure such as [Azure SQL](https://azure.microsoft.com/en-us/products/azure-sql/), [Azure Database by MySQL](https://azure.microsoft.com/en-us/services/mysql/), [Azure Database for PostGres](https://azure.microsoft.com/en-us/services/postgresql/) or [CosmosDB](https://azure.microsoft.com/en-us/services/cosmos-db/).
* **Storage solutions in AKS**: If a managed database doesn't meet your application needs, AKS has multiple storage options available to store consistent data:
  * **Disk-based solutions**: Disks, or block storage, are ideal to store data directly on a raw block-based device. This is ideal to store data for databases you would host in your Kubernetes cluster. In Azure, managed disks are the solution to get block-based storage.
    * Consider whether you want to use a [static disk created outside of AKS](https://docs.microsoft.com/en-us/azure/aks/azure-disk-volume), or if you want [AKS to create the disk on your behalf](https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv).
    * Consider which [Azure disk performance type and size](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-scalability-targets) is required for your workload. 
    * Consider whether you need a [shared disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-shared-enable).
  * **Files-based solutions**: File storage exposes a shared file-system either via NFS or via SMB/CIFS. This is ideal for shared application data and configuration data read by multiple pods in your Kubernetes cluster. Azure has 2 solutions for file based storage: Azure Files and Azure NetApp files.
    * Azure Files:
      * Consider whether you want to use a [static file share created outside of AKS](https://docs.microsoft.com/en-us/azure/aks/azure-files-volume), or if you want [AKS to create the file share dynamically on your behalf](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv).
      * Evaluate if standard performance is sufficient, or if you need premium performance from Azure Files.
      * Evaluate whether you want to use the default SMB/CIFS API to access Azure Files, or if you need NFS support.
      * Consider the network model you want to use to access Azure Files: either via direct pulic IP, via a service endpoint or via privatelink.
    * Azure NetApp Files (ANF):
      * Consider whether you want to use a static ANF share created outside of AKS, or if you want AKS to create the file share dynamically on your behalf via Astra Control.
      * Evaluate which performance tier is required for your workload.
      * Explore the networking recommendations for ANF
  * **Blob**
  * **Other**  
