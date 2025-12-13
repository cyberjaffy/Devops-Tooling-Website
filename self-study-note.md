# Self-Study Notes: Storage Concepts

## 1. NAS (Network-Attached Storage)

NAS is a storage device connected to a network that allows multiple users to access and share data. It is similar to having an external hard drive for an office where all computers can access the same files.

### Protocols Used

* **NFS (Network File System):**
  A protocol that allows computers to access files over a network as if they were stored locally. It is commonly used in Linux and Unix environments.

* **SMB (Server Message Block):**
  A protocol mainly used for sharing files, printers, and other resources across Windows machines.

* **SFTP (Secure File Transfer Protocol):**
  A secure version of FTP used to transfer files over a network. It runs over SSH to provide encryption and security.

---

## 2. SAN (Storage Area Network)

SAN is a specialized, high-speed network that provides **block-level storage**. It allows multiple servers to access shared storage as if it were directly attached to them. Compared to NAS, SAN offers higher performance and is commonly used in enterprise environments.

### Protocol Used

* **iSCSI (Internet Small Computer Systems Interface):**
  A protocol that allows SANs to transmit block-level data over standard TCP/IP networks, enabling long-distance and scalable storage solutions.

---

## 3. Block-Level Storage

Block-level storage stores data in fixed-size chunks called **blocks**. Each block is treated as an independent unit, providing high flexibility and performance. This type of storage is commonly used for databases, virtual machines, and operating system disks.

### Used by Cloud Providers (Example: AWS)

* **Amazon EBS (Elastic Block Store):**
  Provides block storage that can be attached to EC2 instances and used like a physical hard drive.

---

## 4. Object Storage

Object storage manages data as **objects**, each containing the data itself, metadata, and a unique identifier. It is well-suited for storing large volumes of unstructured data such as images, videos, backups, and logs.

### Example in AWS

* **Amazon S3 (Simple Storage Service):**
  A highly scalable and durable object storage service used for file storage, backups, static content, and data analytics.

---

## 5. Network File System (NFS)

NFS is a file-level protocol used for sharing files across a network. In cloud environments, it enables mounting remote file systems so multiple servers can access the same files simultaneously.

### AWS Example

* **Amazon EFS (Elastic File System):**
  A fully managed NFS service that provides scalable shared file storage for EC2 instances.

---

## Key Differences

### Block Storage vs. Object Storage

* **Block Storage:**
  Best for high-performance workloads such as databases and virtual machines. Each block functions like a disk attached to a server.
* **Object Storage:**
  Ideal for large-scale, unstructured data that does not require frequent updates.

### Block Storage vs. NFS

* **Block Storage:**
  Lower-level storage with higher performance, typically used by a single server.
* **NFS:**
  File-level access designed for sharing data across multiple systems.

---

## AWS Storage Summary

* **Block Storage:** Amazon EBS
* **Object Storage:** Amazon S3
* **Network File System:** Amazon EFS

---

## Summary of Related Protocols

* **NFS:** File sharing across Linux/Unix systems
* **SMB:** File sharing across Windows systems
* **SFTP:** Secure file transfer using SSH
* **iSCSI:** Block-level storage over IP networks

---
