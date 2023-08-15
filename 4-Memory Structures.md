# Memory Structures
## Three Major memory sturctures
- **System Golbal Area (SGA)**: A large, shared memory segment that virtually all oracle processes will access at one point or another
- **Process (or Program) Global Area (PGA)**: Memory that is private to a single process or thread; not accessible form other processes/threads
- **User Global Area (UGA)**: memory associated with your session. Locating in SGA when connecting the database using a shared server. Locating is PGA when connecting the database using a dedicated server.

## PGA and UGA
PGA typically allocated via C runtime calls malloc() or memmap()
### Manual PGA Memory Management
**Not Recommended** 
### Automatic PGA Memory Management
Goal: Manximiza the use of RAM while at the same time not using more RAM than you want

Relative parameters: MEMORY_TARGET, PGA_AGGREGATE_TARGET

- PGA_AGGREGATE_TARGET is a goal of an upper limit. It is not a value that is preallocated when the database is started up.
- The amount of PGA memory a process is allocated is typically a function of the amount of memory aviliable and the number of processes competing for space.
- As the workload on your instance goes up (more concurrent queries, concurrent users), the amount of PGA memory allocated to your work areas will go down. The database will try to keep the sum of all PGA allocations under the threshold set by PGA_AGGREGATE_TARGET.
- PGA_AGGREGATE_TARGET is not a hard limit. The instance will attempt to stay in the bounds, but if it can't, it won't stop processing; rather, it will just be forced to exceed that threshold.

## SGA
The SGA is broken into various pools. Here are the major ones:
- *Shared pool*: containing shared cursors, stored procedures, state objects, dictionary caches, etc.
- *Database buffer cache (block buffers)*: Data blocks read from disk as users query and modify data. Contains the most recently used data blocks.
- *Fixed SGA*: Containing internal housekeeping information regarding the state of the instance and database
- *Redo log buffer*: A circular buffer that contains information regarding changes to the database. These changes are written to the online redo logs on disk, used for database recovery.
- *Java pool*: The Java pool is a fixed amount of memory allocated for the JVM running in the database. The Java pool may be resized online while the database is up and running.
- *Large Pool*: Optional memory area used by shared server connections for session memory, by parallel execution features for message buffers, and by RMAN backup for disk I/O buffers. This pool is resizable online.
- *Streams pool*: This is a pool memory used by data sharing tools such as Oracle GoldenGate, Oracle Streams, and Data Pump. Resizable online.
- *Flashback buffer*: Optional memory area used when Flashback database is enabled. The recovery write process will copy modified locks form the buffer cache to the flashback buffer, which are written to Flashback Database logs on disk.
- *Shared I/O pool*: Used for I/O operations on SecureFile Large Objects. This is used for SecureFile deduplication, encryption, and compression.
- *In-memory area*: Optional memory area that allows tables and partitions to be stored in a columnar format.
- *Memoptimize pool*: Optional memory component that optimizes key-based queries.
- *Optional Database Smart Flash Cache*: Optional memory extension to the database buffer cache for Linux and Solaris systems. Resides on solid-state storage devices that use flash memory.

### Granule
The smallest unit of allocation in SGA. A single granule is an area of memory of 4MB, 8MB or 16MB in size.

### Database block buffer cache
The block buffer cache is where Oracle stores database blocks before writing them to disk and after reading them from disk. We have three places to store cached blocks form individual segments in the SGA:
- *Default pool* (hot)
- *Keep pool* (warm)
- *Recyle pool* (do ont care to cache)

These pools were generally regarded as a very fine, low-level tuning device.

### Managing blocks in the buffer cache
The blocks in the buffer cache are basically managed in a single place with two different lists pointing at them
- The list of dirty blocks that need to be written by the database block writer.
- A list of nondirty blocks
  
### Multipel block sizes
Each unique block size must hace its own buffer cache area. The default, keep, and recycle pools will only cache blocks of the default size. In order to have a nondefault block size in your database, you need to hace configured a buffer pool to hold them.

### Shared pool
Share data, queries .etc so that multiple sessions don't have to execute same program again and again. Shared pool is managed based on LRU (least recently used) algorithm.

### Large pool
It is used for allocations of large pieces of memory that are bigger than the shared pool is designed to handle. 

Memory allocated in the large pool is managed in a heap, much in the way C manages memory via malloc() and free(). As soon as you free a chunk of memory, it can be used by other processes.

The large pool is used specifically by
- *Shared server connections*, to allocate the UGA region in the SGA
- *Parallel execution of statements*, to allow for the allocationof interprocess message buffers, which are used to coordinate the parallel query servers.
- *backup* for RMAN disk I/O buffers in some cases

### SGA memory management
- For large databases and/or databases in Linux environments, use ASMM for managing your SGA, and use automatic PGA memory management for setting your PGA. In this mode, you set MEMORY_TARGET to zero, and then you set SGA_TARGET and PGA_AGGREGATE_TARGET to nonzero values.
- For small databases and/or databases in non-Linux environments, then use AMM. For this method, you set MEMORY_TARGET to a nonzero value and set SGA_TARGET and PGA_AGGREGATE_TARGET to zero. Oracle will then adjust the memory pool sizes as it sees fit. In this mode, you can optionally set SGA_TARGET and PGA_AGGREGATE_TARGET to nonzero values, and Oracle will use these settings as a minimum amount of memory to allocate to those two values.
- Do not use the antiquated manual memory management approaches for your SGA and PGA (unless you have a very good reason, like you’re running an old version of Oracle and don’t have one of the other memory management choices available).

#### ASMM (Automatic Shared Memory Management)
Most common way of sizing the SGA. Set the SGA_TARGET parameter to the desired SGA size, leaving out the other SGA-related parameters altogether.

#### AMM (Automatic Memory Management)
Set the MEMORY_TARGET, and the database will then decide the optimal SGA size and PGA size.
