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
