# Locking and Latching
## Locking Issuses
### Lost updates
### Pessimistic locking
The pessimistic locking method would be put into action the instant before a user modifies a value on the screen

Pessimistic locking is useful only in a stateful or connected environment—that is, one where your
application has a continual connection to the database and you are the only one using that connection for at least the life of your transaction.
### Optimistic locking
locking up to the point right before the
update is performed

two ways of implementing optimistic locking
- Using a special column that is maintained by a database trigger or application code to tell us the “version” of the record
- Using a checksum or hash that was computed using the original data

### Optimistic or Pessimistic Locking
pessimistic locking works very well in Oracle and has many advantages over optimistic locking. However, it requires a stateful connection to the database, like a client/server connection. This is because locks are not held across connections. This single fact makes pessimistic locking unrealistic in many cases today.

Today, however, optimistic concurrency control is what I would recommend for most applications.

I tend to use the version column approach with a timestamp column. It gives me the extra update information in a long-term sense. Furthermore, it’s less computationally expensive than a hash or checksum.

If I had to add optimistic concurrency controls to a table that was still being used with a pessimistic locking scheme (e.g., the table was accessed in both client/server applications and over the Web), I would opt for the ORA_HASH approach. The reason is that the existing legacy application might not appreciate a
new column appearing.
