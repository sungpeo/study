# Chapter12. File-System Implementation

파일 시스템은 큰 용량의 데이터를 영구저장하는 용도로 사용하는 보조 저장소에 상주한다. 이번 장은 파일 스토리지를 둘러싼 이슈와 가장 흔하게 쓰이는 보조 저장소에 대한 access에 대해 다룰 것이다. 파일 사용 구조화, 디스크 공간 할당, freed 공간 복구, 데이터의 위치 추적, 그리고 다른 OS에서의 보조 저장소 간의 interface를 알아볼 것이다.

Chapter Objectives

- 로컬 파일시스템과 디렉토리 구조의 구현에 대한 디테일
- remote file system 구현
- block allocation와 free-block 알고리즘 그리고 trade-off

## 12.1 File-System Structure

1. 디스크는 제자리에 다시쓰기가 가능하다. 디스크로부터의 하나의 block을 읽고, 수정하고, 덮어쓰는 것이 가능하다.
2. 디스크는 어떤 블럭이라도 직접 접근하기가 가능하다. read-write head가 움직이고 disk to rotate를 기다리기만 하면

10장에서 자세히 살펴봤었다.
I/O 효율과 메모리로의 I/O 전송 속도를 향상시키기 위해 **blocks**이라는 단위를 사용한다.

**File systems** provide efficient and convenient access to the disk by allowing data to be stored, located, and retrieved easily.

The **I/O control** level consists of device drivers and interrupt handlers to transfer information between the main memory and the disk system. A device driver can be thought of as a translator.

The **file-organization module** knows about files and their logical blocks, as well as physical blocks.

Finally, the **logical file system** manages metadata information. Metadata includes all of the file-system structure except the actual data (or contents of the files). ... A **file-control block** (**FCB**) (an **inode** in UNIX file systems) contains information about the file, including ownership, permissions, and location of the file contents.

## 12.2 File-System Implementation

### 12.2.1 Overview

디스크의 파일시스템은 OS를 어떻게 부팅시키는지, blocks의 총합, location of freed blocks, 디렉토리 구조 등의 정보를 포함한다.

디스크에는 아래와 같은 정보들이 관리된다.

- A **boot control block** (per volume) can contain information needed by the system to boot an operating system from that volume. If the disk does not contain an operating system, this block can be empty. It is typically the first block of a volume. In UFS, it is called the **boot block**. In NTFS, it is the **partition boot sector**.

- A **volume control block** (per volume) contains volume (or partition) details, such as the number of blocks in the partition, the size of the blocks, a free-block count and free-block pointers, and a free-FCB count and FCB pointers. In UFS, this is called a **superblock**. In NTFS, it is stored in the **master file table**.

- A directory structure (per file system) is used to organize the files. In UFS, this includes file names and associated inode numbers. In NTFS, it is stored in the master file table.

- A per-file FCB contains many details about the file. It has a unique identifier number to allow association with a directory entry. In NTFS, this information is actually stored within the master file table, which uses a relational database structure, with a row per file.

메모리에 관리되는 정보는 파일시스템 관리와 성능향상을 위한 캐싱이 있다.

- An in-memory **mount table** contains information about each mounted volume.

- An in-memory directory-structure cache holds the directory information of recently accessed directories. (For directories at which volumes are mounted, it can contain a pointer to the volume table.)

- The **system-wide open-file table** contains a copy of the FCB of each open file, as well as other information

- The **per-process open-file table** contains a pointer to the appropriate entry in the system-wide open-file table, as well as other information.

- Buffers hold file-system blocks when they are being read from disk or written to disk.

...

### 12.2.2 Partitions and Mounting

Each partition can be either “raw,” containing no file system, or “cooked,” containing a file system. **Raw disk** is used where no file system is appropriate. //UNIX swap space, database

This **boot loader** in turn knows enough about the file-system structure to be able to find and load the kernel and start it executing. It can contain more than the instructions for how to boot a specific operating system. For instance, many systems can be dual-booted, allowing us to install multiple operating systems on a single system.

The **root partition**, which contains the operating-system kernel and sometimes other system files, is mounted at boot time. Other volumes can be automatically mounted at boot or manually mounted later, depending on the operating system.

//partition and RAID

### 12.2.3 Virtual File Systems

Data structures and procedures are used to isolate the basic system call functionality from the implementation details. Thus, the file-system implementation consists of three major layers, as depicted schematically in Figure 12.4. The first layer is the file-system interface, based on the open(), read(), write(), and close() calls and on file descriptors.

The second layer is called the **virtual file system** (**VFS**) layer. The VFS layer serves two important functions:

1. It separates file-system-generic operations from their implementation by defining a clean VFS interface. Several implementations for the VFS interface may coexist on the same machine, allowing transparent access to different types of file systems mounted locally.

2. It provides a mechanism for uniquely representing a file throughout a network. The VFS is based on a file-representation structure, called a vnode, that contains a numerical designator for a network-wide unique file. (UNIX inodes are unique within only a single file system.) This network-wide uniqueness is required for support of network file systems. The kernel maintains one vnode structure for each active node (file or directory).

## 12.3 Directory Implementation

The selection of directory-allocation and directory-management algorithms significantly affects the efficiency, performance, and reliability of the file system.

### 12.3.1 Linear List

The simplest method of implementing a directory is to use a linear list of file names with pointers to the data blocks.

The real disadvantage of a linear list of directory entries is that finding a file requires a linear search.

### 12.3.2 Hash Table

it can greatly decrease the directory search time. Insertion and deletion are also fairly straightforward, although some provision must be made for collisions.

The major difficulties with a hash table are its generally fixed size and the dependence of the hash function on that size.

Alternatively, we can use a chained-overflow hash table.

## 12.4 Allocation Methods

The main problem is how to allocate space to these files so that disk space is utilized effectively and files can be accessed quickly.

### 12.4.1 Contiguous Allocation

**Contiguous allocation** requires that each file occupy a set of contiguous blocks on the disk. (Figure 12.5)

Contiguous allocation of a file is defined by the disk address and length (in block units) of the first block.

Problems :

- 새로운 파일을 위한 공간을 찾기가 어렵다.

- **dynamic storage-allocation** (Section 8.3) : first fit/best fit/worst fit 중엔 first fit이 빠르고 쓸만함

- **external fragmentation**

파편화를 막기 위해 다른 디스크에 넣었다가 hole들을 줄여나가는 방향으로 다시 복사하는 방법이 있다. 문제는 시간이 많이 걸린다는 것. down time을 offline으로 가지는 것은 product에는 어울리지 않아서 최근엔 on-line defragmentation을 수행할 수 있으나 역시 performance penalty가 상당하다.

- 파일을 생성할때 얼마나 큰 space를 할당해야할지 알 수 없다.

### 12.4.2 Linked Allocation

With **linked allocation**, each file is a linked list of disk blocks. (Firgure 12.6)

당연히 파편화도 없고 파일을 생성하면서 크기를 예측할 필요도 없다.

Consequently, it is inefficient to support a direct-access capability for linked-allocation files.

Another disadvantage is the space required for the pointers. 그래서 **clusters** 단위로 관리하기도 함.

Yet another problem of linked allocation is reliability. Recall that the files are linked together by pointers scattered all over the disk, and consider what would happen if a pointer were lost or damaged.

An important variation on linked allocation is the use of a **file-allocation table** (**FAT**). This simple but efficient method of disk-space allocation was used by the MS-DOS operating system. 디스크 첫번째 볼륨에 테이블을 가지고 있음. block number로 block을 indexed

### 12.4.3 Indexed Allocation

However, in the absence of a FAT, linked allocation cannot support efficient direct access, since the pointers to the blocks are scattered with the blocks themselves all over the disk and must be retrieved in order. **Indexed allocation** solves this problem by bringing all the pointers together into one location: the **index block**.

- 디스크의 한 블록에 파일의 나머지 블록에 대한 포인터를 유지
- 장점은 연결 할당의 문제점인 직접 접근 불가능 문제를 해결
- 단점은 연결 할당헤 비해 공간낭비가 심함

색인 블록 할당 방법

**linked scheme**

- 처음에는 하나의 색인 블록 할당
- 필요시 여러개의 색인 블록을 연결 리스트로 사용

**multilevel index**

- 색인블록을 다른 색인 블록에 연결
- 4096바이트 블록에서 4B 포인터는 1024개의 색인 가능
- 최대 4GB 파일 생성

**combined scheme**

- 유닉스에서 사용하는 방법
- 파일의 inode에 15개의 포인터를 유지
- 15개 중 12개는 직접 데이터 블록을 가리킴
- 13번째는 single indirect, 14번째는 double indirect, 15번째는 triple indirect

### 12.4.4 Performance

디스크 공간할당 방법의 성능이 고려해야할 내용
- 저장 공간 효율
- 접근속도측면에서 고려해야함

접급근속도 측면
- 연속핟당이 가장 우수
- 파일의 크기 변화에 대처하기 어렵고 가용공간 할당이 어려움

파일 j번째 블록을 읽기위해 읽는 블록의 수
- 연속 할당: 1
- 연결 할당: j-1
- 색인 할당: i+1 (i는 j블록을 읽기 위해 읽는 색인 블록의 수)

## 12.5 Free-Space Management

To keep track of free disk space, the system maintains a **free-space list**. The free-space list records all free disk blocks—those not allocated to some file or directory.

### 12.5.1 Bit Vector

Frequently, the free-space list is implemented as a **bit map** or **bit vector**. Each block is represented by 1 bit. If the block is free, the bit is 1; if the block is allocated, the bit is 0.

advantage : simplicity and efficiency

disadvantage : Given that disk size constantly increases, the problem with bit vectors will continue to escalate as well. (Main memory에 있어야 성능이 나오므로)

### 12.5.2 Linked List

Another approach to free-space management is to link together all the free disk blocks, keeping a pointer to the first free block in a special location on the disk and caching it in memory.
This first block contains a pointer to the next free disk block, and so on.

free list 확인하는 게 느림. 그러나 자주 수행하는 action은 아님.

### 12.5.3 Grouping

A modification of the free-list approach stores the addresses of n free blocks in the first free block. The first n−1 of these blocks are actually free. The last block contains the addresses of another n free blocks, and so on. The addresses of a large number of free blocks can now be found quickly, unlike the situation when the standard linked-list approach is used.

### 12.5.4 Counting

first free block and the number (n) of free contiguouse block that follow the first block.
Although each entry requires more space than would a simple disk address, the overall list is shorter

### 12.5.5 Space Maps

Oracle’s **ZFS** file system (found in Solaris and other operating systems) was designed to encompass huge numbers of files, directories, and even file systems (in ZFS,we can create file-system hierarchies).

...

## 12.6 Efficiency and Performance

Now that we have discussed various block-allocation and directory-management options, we can further consider their effect on performance and efficient disk use.

### 12.6.1 Efficiency

The efficient use of disk space depends heavily on the disk-allocation and directory algorithms in use. For instance, UNIX inodes are preallocated on a volume. Even an empty disk has a percentage of its space lost to inodes. However, by preallocating the inodes and spreading them across the volume, we impropve the file system's performance.

//이런 에시들.. pointer size의 영향과 정하기 어려움

### 12.6.2 Performance

Some systems maintain a separate section of main memory for a **buffer cache**, where blocks are kept under the assumption that they will be used again shortly. Other systems cache file data using a **page cache**. ... Several systems—including Solaris, Linux, and Windows —use page caching to cache both process pages and file data. This is known as **unified virtual memory**. (Figure 12.12)

The memory-mapping call, however, requires using two caches—the page cache and the buffer cache.

Some systems optimize their page cache by using different replacement 
algorithms, depending on the access type of the file. A file being read or
written sequentially should not have its pages replaced in LRU order, because
the most recently used page will be used last, or perhaps never again. Instead,
sequential access can be optimized by techniques known as free-behind and
read-ahead. **Free-behind** removes a page from the buffer as soon as the next
page is requested. The previous pages are not likely to be used again and
waste buffer space.With **read-ahead**, a requested page and several subsequent
pages are read and cached. These pages are likely to be requested after the
current page is processed. Retrieving these data from the disk in one transfer
and caching them saves a considerable amount of time. One might think that
a track cache on the controller would eliminate the need for read-ahead on a
multiprogrammed system. However, because of the high latency and overhead
involved in making many small transfers fromthe

## 12.7 Recovery

### 12.7.1 Consistency Checking

The consistency checker—a systems program such as fsck in UNIX— compares the data in the directory structure with the data blocks on disk and tries to fix any inconsistencies it finds.

### 12.7.2 Log-Structured File Systems

Computer scientists often find that algorithms and technologies originally used in one area are equally useful in other areas. Such is the case with the database log-based recovery algorithms. These logging algorithms have been applied successfully to the problem of consistency checking. The resulting implementations are known as **log-based transaction-oriented** (or **journaling**) file systems.

If the system crashes, the log file will contain zero or more transactions. Any transactions it contains were not completed to the file system, even though they were committed by the operating system, so they must now be completed.

### 12.7.3 Other Solutions

Another alternative to consistency checking is employed by Network Appliance’s WAFL file system and the Solaris ZFS file system. These systems never overwrite blocks with new data. Rather, a transaction writes all data and metadata changes to new blocks. When the transaction is complete, the metadata structures that pointed to the old versions of these blocks are updated to point to the new blocks.

### 12.7.4 Backup and Restore

A typical backup schedule may then be as follows:

- Day 1. Copy to a backup medium all files from the disk. This is called a **full backup**.
- Day 2. Copy to another medium all files changed since day 1. This is an **incremental backup**.
- Day 3. Copy to another medium all files changed since day 2.
...

A user may notice that a particular file is missing or corrupted long after the damage was done. For this reason, we usually plan to take a full backup from time to time that will be saved “forever.” It is a good idea to store these permanent backups far away  from the regular backups to protect against hazard, such as a fire that destroys the computer and all the backups too.

## 12.8 NFS

Network file systems

### 12.8.1 Overview

NFS views a set of interconnected workstations as a set of independentmachines with independent file systems. The goal is to allow some degree of sharing among these file systems (on explicit request) in a transparent manner.

The NFS specification distinguishes between the services provided by a mount mechanism and the actual remote-file-access services. Accordingly, two separate protocols are specified for these services: a mount protocol and a protocol for remote file accesses, the NFS protocol. The protocols are specified as sets of RPCs. These RPCs are the building blocks used to implement transparent remote file access.

### 12.8.2 The Mount Protocol

The **mount protocol** establishes the initial logical connection between a server and a client.

The server maintains an export list that specifies local file systems that it exports for mounting, along with names of machines that are permitted to mount them.

### 12.8.3 The NFS Protocol

The NFS protocol provides a set of RPCs for remote file operations. The procedures support the following operations:

- Searching for a file within a directory
- Reading a set of directory entries
- Manipulating links and directories
- Accessing file attributes
- Reading and writing files

NFS is integrated into the operating system via a VFS. As an illustration of the architecture, let’s trace how an operation on an already-open remote file is handled (follow the example in Figure 12.15).

### 12.8.4 Path-Name Translation

**Path-name translation** in NFS involves the parsing of a path name such as /usr/local/dir1/file.txt into separate directory entries, or components:
(1) usr, (2) local, and (3) dir1.

So that lookup is fast, a directory-name-lookup cache on the client side holds the vnodes for remote directory names. This cache speeds up references to files with the same initial path name.

### 12.8.5 Remote Operations

Tuning the system for performance makes it difficult to characterize the consistency semantics of NFS. New files created on a machine may not be visible elsewhere for 30 seconds. Furthermore, writes to a file at one site may or may not be visible at other sites that have this file open for reading. New opens of a file observe only the changes that have already been flushed to the server. Thus, NFS provides neither strict emulation of UNIX semantics nor the session semantics of Andrew (Section 11.5.3.2). In spite of these drawbacks, the utility and good performance of the mechanism make it the most widely used multi-vendor-distributed system in operation.

## 12.9 Example: The WAFL File System

## 12.10 Summary
