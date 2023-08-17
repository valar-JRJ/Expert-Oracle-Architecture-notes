# Concurrency and Meltiversioning
Oracle uses a varity of locks, including
- *TX(transaction) locks*: these locks are acquried for the duration of a data-modifying transaction
- *TM(DML enqueue) and DDL locks*: these locks ensure that the structure of an object is not altered while you are modifying its contents (TM lock) or the object itself (DDL lock)
- Latches and mutexes: these are internal locks that Oracle employs to mediate access to its shared data structures.
## Transaction Isolation Levels
There are four levels of transaction isolation, with different possible outcomes for the same transaction senario. These isolation levels are defined in terms of three phenomena that are either permitted or not at a given isolation level:
- Dirty read: you are permitted to read uncommitted, or dirty data. Data integrity is compromised, foreign keys are violated, and unique constraints are ignored.
- Nonrepeatable read: This simply means that if you read a row at time T1 and attempt to reread that row at time T2, the row may have changed, or it may have disappeared, or it may have been updated, and so on.
- Phantom read: This means that if you execute a query at time T1 and reexecute it at time T2, additional rows may have been added to the database, which will affect your results.

| Isolation Level   | Dirty Read | Nonrepeatable Read | Phantom Read |
|-------------------|------------|--------------------|--------------|
| READ UNCOMMITTED  | Permitted  | Permitted          | Permitted    |
| READ COMMITTED    | –          | Permitted          | Permitted    |
| REPEATABLE READ   | –          | –                  | Permitted    |
| SERIALIZABLE      | –          | –                  | –            |

### READ UNCOMITTED
READ UNCOMMITTED隔离级别允许脏读。Oracle没有使用脏读这样的机制，它甚至不允许脏读。READ UNCOMMITTED的根本目标是提供一个基于标准的定义以支持非阻塞读，而Oracle会默认支持非阻塞读。

### Read Committed
READ COMMITTED是指事务只能读取数据库中已经提交的数据。这里没有脏读，不过可能有不可重复读和幻读。这是Oracle数据库的默认模式。

### REPEATABLE READ
REPEATABLE READ的目标是不仅能给出一致的正确答案，还能避免丢失更新。在Oracle中通过多版本控制，得到的答案相对于查询开始执行的那个时间点是一致的。

### SERIALIZABLE
在Serializable隔离级别下，事务之间的相互干扰最小。一个事务的修改不会被其他事务看到，直到该事务提交。其他事务在等待一个事务完成后才能访问被该事务修改过的数据。Oracle实现SERIALIZABLE事务的方式很简单，将原本语句级的读一致性扩展到事务级。在这个隔离级别下，每个查询返回的结果并非是语句开始的那个时间点作为结果一致性的基准点，这个基准点是在事务开始那一刻。

## Oracle多版本控制
Oracle数据库的多版本控制（Multi-Version Concurrency Control，MVCC）是一种用于实现并发事务处理的机制。它允许多个事务在同一时间访问数据库，而不会出现数据不一致或冲突的问题。MVCC在Oracle数据库中通过使用不同版本的数据来实现。

MVCC的核心思想是每个事务在开始时都能看到一致的数据快照，而不会被其他事务的修改所影响。以下是MVCC的一般工作流程：

1. 读取操作：当事务要读取数据时，它会获取数据的一个一致性视图（Consistent Read View），这个视图反映了事务开始时数据库中的数据状态。这意味着事务看到的数据是在它开始时存在的，而不会看到其他正在进行的事务的未提交更改。

2. 写入操作：当事务要修改数据时，数据库会为该修改创建一个新的版本，并将该版本标记为该事务的“私有”版本。其他事务仍然可以在自己的一致性视图下继续读取旧版本的数据。

3. 数据一致性：当事务提交时，它的所有修改都会被提交到数据库，并且将这些修改标记为已提交。其他事务的一致性视图会相应地更新，以反映已提交的更改。

**MVCC的优点包括：**

- 高并发性：多个事务可以同时读取数据，而不会互相干扰，从而提高了数据库的并发处理能力。
- 无阻塞读取：读取操作不会被写入操作阻塞，因为读取操作可以在事务的一致性视图下继续进行。
- 数据一致性：每个事务都在自己的一致性视图下运行，保证了事务间的隔离性，防止脏读、不可重复读等问题。

需要注意的是，MVCC并非适用于所有情况，特别是在需要强制数据一致性的情况下，可能需要额外的操作来确保数据的准确性。此外，MVCC也可能会增加一些存储开销，因为需要维护多个数据版本。

## Oracle的读一致性
Oracle数据库通过使用 undo（撤销）机制来实现读一致性，确保在事务执行期间其他事务对数据的修改不会影响正在运行的事务的一致性读取。以下是Oracle数据库如何通过 undo 实现读一致性的工作流程：

- 事务开始： 当一个事务开始时，Oracle数据库会为该事务创建一个一致性视图，即事务开始时数据库的快照。
- 一致性视图的生成： 事务的一致性视图是通过查询 undo 记录来生成的。这些 undo 记录包含了在事务开始后发生的对数据的修改。
- 数据读取： 当事务要读取数据时，它会在其一致性视图下查询数据库。数据库会根据一致性视图和 undo 记录，返回事务开始时的数据版本。
- 无冲突读取： 由于事务的一致性视图反映了事务开始时的数据库状态，其他正在进行的事务的修改不会影响当前事务的读取。即使其他事务在事务开始后修改了相同的数据，这些修改也不会反映在当前事务的读取结果中。
- 事务提交或回滚： 当事务完成操作并提交时，数据库会根据提交的修改将新数据版本写入数据文件。如果事务回滚，数据库会使用 undo 记录撤销事务所做的修改。

通过这种方式，Oracle数据库确保了在一个事务的执行过程中，其他事务对数据的修改不会影响当前事务的读取操作，从而实现了读一致性。这是多版本并发控制（MVCC）的关键部分，允许多个事务并发地访问数据库，同时保持数据的一致性和完整性

## 写一致性
一致读：”发现“要修改的行 

当前读： 取得数据块来进行真正的修改。

Oracle 会用where子句中的列加上行触发器中的列，来比较如果有不同就会重启修改。