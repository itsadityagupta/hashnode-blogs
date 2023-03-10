---
title: "Hadoop Distributed File System"
datePublished: Sat Dec 17 2022 08:27:08 GMT+0000 (Coordinated Universal Time)
cuid: clbroek2e000j08k1ewgnfjdy
slug: hdfs-architecture
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1671265467676/iDYuVb6wo.png
tags: hadoop, big-data, hdfs

---

## Introduction

HDFS is a distributed file system to run on commodity hardware. It is highly fault-tolerant and is designed to be deployed on low-cost hardware, providing high throughput access to application data and is suitable for applications that have large data sets.

## Assumptions and Goals of HDFS

1.  **Hardware failure is the norm rather than the exception.**
    
    Any running HDFS servers can go down at any time, making it an essential goal for HDFS to be fault tolerant and recover automatically and quickly.
    
2.  **Applications running on HDFS need streaming access to the data.**
    
    HDFS is designed more for batch processing rather than interactive use by users. The emphasis is on the high throughput of data access rather than the low latency of data access.
    
3.  **Applications running on HDFS have large data sets.**
    
    HDFS should provide high aggregate data bandwidth and scale to hundreds of nodes in a single cluster. It should support tens of millions of files in a single instance.
    
4.  **HDFS applications need a write-once-read-many access model for files.**
    
    A file once created, written, and closed need not be changed except for appends and truncates. Appending the content to the end of the files is supported but cannot be updated at an arbitrary point.
    
    This assumption simplifies data coherency issues and enables high throughput data access. (A MapReduce application or a web crawler application fits perfectly with this model.)
    
5.  **Moving computation is cheaper than moving data.**
    
    It is often better to migrate the computation closer to where the data is located rather than moving the data to where the application is running. This minimizes network congestion and increases the overall throughput of the system.
    
6.  **Portability across heterogeneous hardware and software platforms.**
    
    HDFS has been designed to be easily portable from one platform to another. This facilitates the widespread adoption of HDFS as a platform of choice for a large set of applications.
    

## Namenode and Datanodes

HDFS follows a master/slave architecture. An HDFS cluster consists of a single NameNode, a master server that manages the file system namespace and regulates access to files by clients. In addition, there are a number of DataNodes, usually, one per node in the cluster, which manage storage attached to the nodes that they run on.

HDFS exposes a file system namespace and allows user data to be stored in files. Internally, a file is split into one or more blocks and these blocks are stored in a set of DataNodes.

The NameNode executes file system namespace operations like opening, closing, and renaming files and directories. It also determines the mapping of blocks to DataNodes. The DataNodes are responsible for serving read and write requests from the file system’s clients. The DataNodes also perform block creation, deletion, and replication upon instruction from the NameNode.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670675221872/iglDXAnAl.png align="center")

The existence of a single NameNode in a cluster greatly simplifies the architecture of the system. The NameNode is the arbitrator and repository for all HDFS metadata. The system is designed in such a way that user data never flows through the NameNode.

An application can specify the number of replicas of a file that should be maintained by HDFS. The number of copies of a file is called the replication factor of that file. This information is stored by the NameNode.

## Data Replication

HDFS stores each file as a sequence of blocks and these blocks are replicated across the datanodes for fault tolerance.

> Though the block size and the replication factor are configurable per file, the default size of a block is 128 MB in Hadoop 2.0 whereas it is 64 MB in Hadoop 1.0.

All blocks in a file except the last block are of the same size.

Namenode makes all decisions regarding the replication of blocks. It periodically receives *heartbeats* and **blockreports** from datanodes (default = 3 sec) in the cluster. The receipt of a heartbeat implies that the DataNode is functioning properly.

A Blockreport contains a list of all the blocks on a DataNode. Here's a sample block report:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1670592868993/WVXzYCmWY.PNG align="center")

### Replica Placement in HDFS

HDFS uses a rack-aware replica placement policy. The purpose of using such a policy is to improve data reliability, availability, and network bandwidth utilization.

HDFS clusters run on a large number of computers spread across multiple racks. Communication between two nodes in different racks has to go through switches. In most cases, the network bandwidth between nodes in the same rack is greater than the bandwidth between the nodes in different racks.

The Namenode determines the rack id each Datanode belongs to via a process called ["Hadoop Rack Awareness"](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/RackAwareness.html).

For a default replication factor of 3, the HDFS replication policy is as follows:

If the writer is on the datanode, place the replica in that same machine. Otherwise, place it in any random datanode in the same rack as that of the writer. Now place another replica in a node on a different (remote) rack and the last replica on a different node in the same remote rack.

This policy cuts the inter-rack write traffic which generally improves write performance without impacting data reliability and availability guarantees.

If the replication factor is greater than 3, the placement of the 4th and following replicas are determined randomly while keeping the number of replicas per rack below the upper limit (which is (replicas - 1) / racks + 2).

Additionally, HDFS supports 4 different pluggable [Block Placement Policies](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsBlockPlacementPolicies.html). Users can choose the policy based on their infrastructure and use case. By default HDFS supports *BlockPlacementPolicyDefault*.

### Replica Selection in HDFS

To minimize global bandwidth consumption and read latency, HDFS tries to satisfy a read request from a replica that is closest to the reader. If a replica exists on the same rack as the reader node, then that is preferred to satisfy the read request.

If an HDFS cluster spans multiple data centres, then a replica that is resident in the local data centre is preferred over any remote replica.

### Safemode

On startup, the NameNode enters a special state called Safemode. Replication of data blocks does not occur when the NameNode is in the Safemode state.

Each block has a specified minimum number of replicas. A block is considered safely replicated when the minimum number of replicas of that data block has checked in with the NameNode.

After a configurable percentage of safely replicated data blocks checks in with the NameNode (plus an additional 30 seconds), the NameNode exits the Safemode state. It then determines the list of data blocks (if any) that still have fewer than the specified number of replicas. The NameNode then replicates these blocks to other DataNodes.

## The Persistence of File System Metadata

The NameNode uses a transaction log called the **EditLog** to persistently record every change that occurs to file system metadata. For example, creating a new file in HDFS causes the NameNode to insert a record into the EditLog indicating this. Similarly, changing the replication factor of a file causes a new record to be inserted into the EditLog. The NameNode uses a file in its local host OS file system to store the EditLog.

The entire file system namespace, including the mapping of blocks to files and file system properties, is stored in a file called the FsImage. The FsImage is stored as a file in the NameNode’s local file system too.

When the NameNode starts up, or a checkpoint is triggered by a configurable threshold, it reads the FsImage and EditLog from disk, applies all the transactions from the EditLog to the in-memory representation of the FsImage, and flushes out this new version into a new FsImage on disk. It can then truncate the old EditLog because its transactions have been applied to the persistent FsImage. This process is called a checkpoint.

The purpose of a checkpoint is to make sure that HDFS has a consistent view of the file system metadata by taking a snapshot of the file system metadata and saving it to FsImage.

Even though it is efficient to read a FsImage, it is not efficient to make incremental edits directly to a FsImage. Instead of modifying FsImage for each edit, we persist the edits in the Editlog. During the checkpoint, the changes from Editlog are applied to the FsImage.

A checkpoint can be triggered at a given time interval (dfs.namenode.checkpoint.period) expressed in seconds, or after a given number of filesystem transactions have accumulated (dfs.namenode.checkpoint.txns). If both of these properties are set, the first threshold to be reached triggers a checkpoint.

On the other hand, the DataNode has no knowledge of HDFS files. It stores each block of HDFS data in a separate file in its local file system. The DataNode does not create all files in the same directory. Instead, it uses a heuristic to determine the optimal number of files per directory and creates subdirectories appropriately.

It is not optimal to create all local files in the same directory because the local file system might not be able to efficiently support a huge number of files in a single directory.

When a DataNode starts up, it scans through its local file system, generates a list of all HDFS data blocks that correspond to each of these local files, and sends this report to the NameNode. The report is called the *Blockreport*.

## The Communication Protocols

All HDFS communication protocols are layered on top of the TCP/IP protocol. A client establishes a connection to a configurable TCP port on the NameNode machine. It talks the ClientProtocol with the NameNode. The DataNodes talk to the NameNode using the DataNode Protocol. A Remote Procedure Call (RPC) abstraction wraps both the Client Protocol and the DataNode Protocol. By design, the NameNode never initiates any RPCs. Instead, it only responds to RPC requests issued by DataNodes or clients.

## Robustness

The primary objective of HDFS is to store data reliably even in the presence of failures. The three common types of failures are NameNode failures, DataNode failures and network partitions.

### **Data Disk Failure, Heartbeats and Re-Replication**

A network partition can cause a subset of DataNodes to lose connectivity with the NameNode. The NameNode detects this condition by the absence of a Heartbeat message. The NameNode marks DataNodes without recent Heartbeats as dead and does not forward any new IO requests to them.

Any data that was registered to a dead DataNode is not available to HDFS anymore. DataNode death may cause the replication factor of some blocks to fall below their specified value. The NameNode constantly tracks which blocks need to be replicated and initiates replication whenever necessary.

The necessity for re-replication may arise due to many reasons:

*   a DataNode may become unavailable,
    
*   a replica may become corrupted,
    
*   a hard disk on a DataNode may fail, or
    
*   the replication factor of a file may be increased.
    

The time-out to mark DataNodes dead is conservatively long (over 10 minutes by default) in order to avoid a replication storm caused by state flapping of DataNodes. Users can set shorter intervals to mark DataNodes as stale and avoid stale nodes on reading and/or writing by configuration for performance-sensitive workloads.

### **Cluster Rebalancing**

The HDFS architecture is compatible with data rebalancing schemes. A scheme might automatically move data from one DataNode to another if the free space on a DataNode falls below a certain threshold. In the event of a sudden high demand for a particular file, a scheme might dynamically create additional replicas and rebalance other data in the cluster. These types of data rebalancing schemes are not yet implemented.

### **Data Integrity**

It is possible that a block of data fetched from a DataNode arrives corrupted. This corruption can occur because of faults in a storage device, network faults, or buggy software. The HDFS client software implements checksum checking on the contents of HDFS files.

When a client creates an HDFS file, it computes a checksum of each block of the file and stores these checksums in a separate hidden file in the same HDFS namespace. When a client retrieves file contents it verifies that the data it received from each DataNode matches the checksum stored in the associated checksum file. If not, then the client can opt to retrieve that block from another DataNode that has a replica of that block.

### **Metadata Disk Failure**

The FsImage and the EditLog are central data structures of HDFS. A corruption of these files can cause the HDFS instance to be non-functional. For this reason, the NameNode can be configured to support maintaining multiple copies of the FsImage and EditLog.

Any update to either the FsImage or EditLog causes each of the FsImages and EditLogs to get updated synchronously. This synchronous updating of multiple copies of the FsImage and EditLog may degrade the rate of namespace transactions per second that a NameNode can support. However, this degradation is acceptable because even though HDFS applications are very data-intensive in nature, they are not metadata intensive. When a NameNode restarts, it selects the latest consistent FsImage and EditLog to use.

Another option to increase resilience against failures is to enable high availability using multiple NameNodes either with a [shared storage on NFS](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithNFS.html) or using a [distributed edit log](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html) (called Journal). The latter is the recommended approach.

### Snapshots

[Snapshots](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html) support storing a copy of data at a particular instant in time. One usage of the snapshot feature may be to roll back a corrupted HDFS instance to a previously known good point in time.

## Data Organisation

### Data Blocks

Applications that are compatible with HDFS are those that deal with large data sets. These applications write their data only once but they read it one or more times and require these reads to be satisfied at streaming speeds. HDFS supports write-once-read-many semantics on files.

A typical block size used by HDFS is 128 MB. Thus, an HDFS file is chopped up into 128 MB chunks, and if possible, each chunk will reside on a different DataNode.

### Replication Pipelining

When a client is writing data to an HDFS file with a replication factor of three, the NameNode retrieves a list of DataNodes using a replication target-choosing algorithm. This list contains the DataNodes that will host a replica of that block.

The client then writes to the first DataNode. The first DataNode starts receiving the data in portions, writes each portion to its local repository and transfers that portion to the second DataNode in the list. The second DataNode, in turn, starts receiving each portion of the data block, writes that portion to its repository and then flushes that portion to the third DataNode. Finally, the third DataNode writes the data to its local repository.

Thus, a DataNode can be receiving data from the previous one in the pipeline and at the same time forward data to the next one in the pipeline. Thus, the data is pipelined from one DataNode to the next.

## Accessibility

HDFS can be accessed from applications in many different ways.

### **FS Shell**

HDFS allows user data to be organized in the form of files and directories. It provides a command line interface called [FS shell](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html) that lets a user interact with the data in HDFS. The syntax of this command set is similar to other shells (e.g. bash, csh) that users are already familiar with.

Here are some sample action/command pairs:

| **Action** | **Command** |
| --- | --- |
| Create a directory named /foodir | bin/hadoop dfs -mkdir /foodir |
| Remove a directory named /foodir | bin/hadoop fs -rm -R /foodir |
| View the contents of a file named /foodir/myfile.txt | bin/hadoop dfs -cat /foodir/myfile.txt |

### **DFSAdmin**

The DFSAdmin command set is used for administering an HDFS cluster. These are commands that are used only by an HDFS administrator. Here are some sample action/command pairs:

| **Action** | **Command** |
| --- | --- |
| Put the cluster in Safemode | bin/hdfs dfsadmin -safemode enter |
| Generate a list of DataNodes | bin/hdfs dfsadmin -report |
| Recommission or decommission DataNode(s) | bin/hdfs dfsadmin -refreshNodes |

### **Browser Interface**

A typical HDFS install configures a web server to expose the HDFS namespace through a configurable TCP port. This allows a user to navigate the HDFS namespace and view the contents of its files using a web browser.

## Space Reclamation

### **File Deletes and Undeletes**

If trash configuration is enabled, files removed by [FS Shell](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#rm) are not immediately removed from HDFS. Instead, HDFS moves it to a trash directory (each user has its own trash directory under `/user/<username>/.Trash`). The file can be restored quickly as long as it remains in the trash.

Most recent deleted files are moved to the current trash directory (`/user/<username>/.Trash/Current)`, and in a configurable interval, HDFS creates checkpoints (under `/user/<username>/.Trash/<date>`) for files in the current trash directory and deletes old checkpoints when they are expired. See [expunge command of FS shell](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#expunge) about checkpointing of trash.

After the expiry of its life in the trash, the NameNode deletes the file from the HDFS namespace. The deletion of a file causes the blocks associated with the file to be freed.

*Note that there could be an appreciable time delay between the time a file is deleted by a user and the time of the corresponding increase in free space in HDFS.*

### **Decrease Replication Factor**

When the replication factor of a file is reduced, the NameNode selects excess replicas that can be deleted. The next Heartbeat transfers this information to the DataNode. The DataNode then removes the corresponding blocks and the corresponding free space appears in the cluster.

Once again, there might be a time delay between the completion of the setReplication API call and the appearance of free space in the cluster.

## Credits

*   [HDFS Documentation](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
    

* * *