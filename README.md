# ES-storage

Kubernetes can be used for both stateless and stateful workloads. Stateful workloads need a storage solution to store their state onto. Within the Azure Kubernetes Service (AKS) multiple storage solutions exist. Picking the right solution for your storage needs can be a challenge, and the goal of this document is to provide you with guidance in making the right choice.

Multiple storage solutions exist for AKS. Solutions range from managed databases, to disks, files and blob storage. Within each solution there are also different options to consider. In the design considerations and design recommendations sections you'll learn more about the different solutions and options available. 

# How to select the right storage service
Picking the right storage solution can be hard, and will require some testing on your side. However, consider the following high level guidelines when selecting a storage solution in AKS:
* For structured application data, a managed datastore is recommended.
* For high performance storage that is going to be used by a database platform running on Kubernetes, leverage disks. Consider using either premium or ultra disks for the best performance.
* For unstructured data like photos, videos and text documents leverage blob storage. Consider reading/writing to blob directly from within your application.
* For shared configuration data with limited performance requirement, leverage Azure Files standard.
* For shared application data with high performance requirements, leverage either Azure Files premium or Azure NetApp Files. Ensure your instance has sufficient network bandwidth to handle application requests and storage requests (given SMB or NFS traffic goes over the network stack).
   
# Design considerations
The following is a list of design considerations when it comes to storage in AKS. Consider where storage is required in your AKS environment, and determine the best solution for each requirement.

* **Operating system disks**: Each virtual machine (VM) in Azure requires a disk for its operating system. Given Kubernetes nodes are ephemeral, AKS defaults to using [ephemeral OS drives](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#ephemeral-os) on supported VM sizes.
  * If required, it is possible to use regular managed disks instead for the nodes in your AKS cluster, in case you need to persist certain data that you store on the OS drive.
  * If you select a [managed disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types) as the operating system disk, ensure it is sized appropriately to support the requirements of the operating system, kubernetes system and your workload. 
* **Application data**: Some applications need a consistent data store to store application data. If a database is required, consider exploring the managed database options in Azure such as [Azure SQL](https://azure.microsoft.com/en-us/products/azure-sql/), [Azure Database by MySQL](https://azure.microsoft.com/en-us/services/mysql/), [Azure Database for PostGres](https://azure.microsoft.com/en-us/services/postgresql/) or [CosmosDB](https://azure.microsoft.com/en-us/services/cosmos-db/).
* **Storage solutions in AKS**: If a managed database doesn't meet your application needs, AKS has multiple storage options available to store consistent data:
  * **Disk-based solutions**: Disks, or block storage, are ideal to store data directly on a raw block-based device. This is ideal to store data for databases you would host in your Kubernetes cluster. In Azure, managed disks are the solution to get block-based storage.
    * Consider whether you want to use a [static disk created outside of AKS](https://docs.microsoft.com/en-us/azure/aks/azure-disk-volume), or if you want [AKS to dynamically create the disk on your behalf](https://docs.microsoft.com/en-us/azure/aks/azure-disks-dynamic-pv).
    * Consider which [Azure disk performance type and size](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-scalability-targets) is required for your workload. 
    * Consider whether you need a [shared disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-shared-enable).
    * Ensure your [Kubernetes node size](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes) is large enough to support both the amount of disks as well as the aggregate throughput requirements.
    * Ensure your [managed disk](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types) is sized approriately for your workload's performance requirements. For standard HDD, standard SSD and premium v1 disks the performance increases as the disk size increases.
  * **Ephemeral disks solutions**: There are cases where you need either a non-persistent temporary storage location or where you want to leverage the high-performance drives in the [storage-optimized VM-series](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-storage). To connect to an ephemeral volume, you can either use the `emptydir` [option](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) in Kubernetes, or use the [CSI ephemeral local volume](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/#csi-ephemeral-volumes) driver. `emptydir` is recommended for ephemeral data such as a scratch space, whereas the CSI ephemeral local volume is recommended for storage on the storage-optimzed VM-series.
  * **Files-based solutions**: File storage exposes a shared file-system either via NFS or via SMB/CIFS. This is ideal for shared application data and configuration data read by multiple pods in your Kubernetes cluster. Azure has 2 solutions for file based storage: Azure Files and Azure NetApp files.
    * Azure Files:
      * Consider whether you want to use a [static file share created outside of AKS](https://docs.microsoft.com/en-us/azure/aks/azure-files-volume), or if you want [AKS to create the file share dynamically on your behalf](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv).
      * Evaluate if standard performance is sufficient, or if you need premium performance from Azure Files.
      * Evaluate whether you want to use the default SMB/CIFS API to access Azure Files, or if you need NFS support.
      * Consider the network model you want to use to access Azure Files: either via direct pulic IP, via a service endpoint or via privatelink.
    * Azure NetApp Files (ANF):
      * Consider whether you want to use a [static ANF share created outside of AKS](https://docs.microsoft.com/en-us/azure/aks/azure-netapp-files#provision-azure-netapp-files-volumes-statically), or if you want [AKS to create the file share dynamically](https://docs.microsoft.com/en-us/azure/aks/azure-netapp-files#provision-azure-netapp-files-volumes-dynamically) on your behalf via Astra Control.
      * Evaluate which performance tier is required for your workload.
      * Explore the [networking recommendations for ANF](https://docs.microsoft.com/en-us/azure/azure-netapp-files/azure-netapp-files-network-topologies).
  * **Blob**: Azure blob storage is Microsoft's object storage platform. It is accessible via an HTTP API or through the SDKs.
    * Evaluate which [data redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy) (defined at storage account level) is required and which [performance tier](https://docs.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview) of blob storage is needed.
    * Consider which [authentication method](https://docs.microsoft.com/en-us/azure/storage/common/authorize-data-access) to blob you want to use: storage key, SAS or AAD-integrated.
    * Typically, applications accessing blob storage make use of the API in the application through [one of the SDKs](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction); making the interaction with blob abstracted from the Kubernetes cluster.
    * If you want to access blob storage like a file system, you can leverage the [blob CSI driver](https://github.com/kubernetes-sigs/blob-csi-driver) in Kubernetes. This driver will allow you to access blob through either an [NFSv3 protocol](https://docs.microsoft.com/en-us/azure/storage/blobs/network-file-system-protocol-support) or through a [fuse driver](https://github.com/Azure/azure-storage-fuse).
  * **Other**: There are a number of specialized storage solutions in Azure that can also be integrated with Kubernetes. These won't be explored in depth in this document.
    * [HPC cache](https://docs.microsoft.com/en-us/azure/aks/azure-hpc-cache): Azure HPC Cache speeds access to your data for high-performance computing (HPC) tasks. By caching files in Azure, Azure HPC Cache brings the scalability of cloud computing to your existing workflow.  
    * [ADLS Gen 2](https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction): A special type of Azure blob storage optimized for big data workloads like Hadoop and Spark.
  

# Design recommendations
* For OS disks, ephemeral disks are recommended. In order to benefit from this feature, make sure to select a VM size with an adequately sized temporary disk.
* For application data, managed databases are recommended.
* For all storage solutions which support this, Private Link is the recommended solution to access storage.
* For Azure disks:
  * In most cases, Premium or Ultra disks are recommended to ensure adequate performance.
  * Ensure your kubernetes node size is large enough to support the amount of disks and aggregate throughput.
  * Avoid striping across multiple disks in Kubernetes.
  * Leverage PVC/PV in Kubernetes to dynamically create disks where required.
* For Azure Files:
  * In case performance is critical, the premium tier is recommended.
  * Have dedicated storage accounts for your file shares.
  * Consider carefully whether you want Kubernetes to create the file shares or if you want to create them statically outside of Kubernetes.
* For Azure Blob:
  * Use an application-level SDK to interface with blob storage.
  * Leverage AAD-integrated authorization for blob storage. Avoid using the shared storage account key.
  * Leverage lifecycle management policies to tier infrequently accesses data to a cooler access tier.
  * If you cannot leverage an application-level SDK to interface with blob, consider using the NFSv3 option in the blob CSI driver.
