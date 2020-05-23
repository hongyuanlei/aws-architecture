## AWS Storage Gateway

AWS Storage Gateway is a service connecting an on-premises software appliance with cloud-
based storage to provide seamless and secure integration between an organization’s on- premises IT environment and AWS storage infrastructure. The service enables you to store data securely on the AWS cloud in a scalable and cost-effective manner. AWS Storage Gateway supports industry-standard storage protocols that work with your existing applications. It provides low-latency performance by caching frequently accessed data on- premises while encrypting and storing all of your data in Amazon S3 or Amazon Glacier.

### Overview

AWS Storage Gateway’s software appliance is available for download as a Virtual Machine (VM) image that you install on a host in your data center and then register with your AWS account through the AWS Management Console. The storage associated with the appliance is exposed as an iSCSI device that can be mounted by your on-premises applications.

There are three configurations for AWS Storage Gateway: Gateway-Cached volumes, Gateway-Stored volumes, and Gateway-Virtual Tape Libraries (VTL).

### Gateway-Cached Volumes

Gateway-Cached volumes allow you to expand your local storage capacity into Amazon S3. All data stored on a Gateway-Cached volume is moved to Amazon S3, while recently read data is retained in local storage to provide low-latency access. While each volume is limited to a maximum size of 32TB, a single gateway can support up to 32 volumes for a maximum storage of 1 PB.
Point-in-time snapshots can be taken to back up your AWS Storage Gateway. These snapshots are performed incrementally, and only the data that has changed since the last snapshot is stored.
All Gateway-Cached volume data and snapshot data is transferred to Amazon S3 over encrypted Secure Sockets Layer (SSL) connections. It is encrypted at rest in Amazon S3 using Server-Side Encryption (SSE). However, you cannot directly access this data with the Amazon S3 API or other tools such as the Amazon S3 console; instead you must access it through the AWS Storage Gateway service.

### Gateway-Stored Volumes

Gateway-Stored volumes allow you to store your data on your on-premises storage and asynchronously back up that data to Amazon S3. This provides low- latency access to all data, while also providing off-site backups taking advantage of the durability of Amazon S3. The data is backed up in the form of Amazon Elastic Block Store (Amazon EBS) snapshots. While each volume is limited to a maximum size of 16TB, a single gateway can support up to 32 volumes for a maximum storage of 512TB.
Similar to Gateway-Cached volumes, you can take snapshots of your Gateway-Stored volumes. The gateway stores these snapshots in Amazon S3 as Amazon EBS snapshots. When you take a new snapshot, only the data that has changed since your last snapshot is stored. You can initiate snapshots on a scheduled or one-time basis. Because these snapshots are stored as Amazon EBS snapshots, you can create a new Amazon EBS volume from a Gateway-Stored volume.
All Gateway-Stored volume data and snapshot data is transferred to Amazon S3 over encrypted SSL connections. It is encrypted at rest in Amazon S3 using SSE. However, you cannot access this data with the Amazon S3 API or other tools such as the Amazon S3 console.
If your on-premises appliance or even entire data center becomes unavailable, the data in AWS Storage Gateway can still be retrieved. If it’s only the appliance that is unavailable, a new appliance can be launched in the data center and attached to the existing AWS Storage Gateway. A new appliance can also be launched in another data center or even on an Amazon EC2 instance on the cloud.

### Gateway Virtual Tape Libraries (VTL) 

Gateway-VTL offers a durable, cost-effective solution to archive your data on the AWS cloud. The VTL interface lets you leverage your existing tape-based backup application infrastructure to store data on virtual tape cartridges that you create on your Gateway-VTL.
A virtual tape is analogous to a physical tape cartridge, except the data is stored on the AWS cloud. Tapes are created blank through the console or programmatically and then filled with backed up data. A gateway can contain up to 1,500 tapes (1 PB) of total tape data. Virtual tapes appear in your gateway’s VTL, a virtualized version of a physical tape library. Virtual tapes are discovered by your backup application using its standard media inventory procedure.
When your tape software ejects a tape, it is archived on a Virtual Tape Shelf (VTS) and stored in Amazon Glacier. You’re allowed 1 VTS per AWS region, but multiple gateways in the same region can share a VTS.

### Use Cases

There are several use cases where AWS Storage Gateway is an excellent choice, including, but not limited to:

- Gateway-Cached volumes enable you to expand local storage hardware to Amazon S3, allowing you to store much more data without drastically increasing your storage hardware or changing your storage processes.

- Gateway-Stored volumes provide seamless, asynchronous, and secure backup of your on- premises storage without new processes or hardware.

- Gateway-VTLs enable you to keep your current tape backup software and processes while storing your data more cost-effectively and simply on the cloud.