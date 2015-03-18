---
layout: post
title: "You're up and running!"
published: true
---

### Optimistic locking VS Select for update

Your developer is mistaken. You need either SELECT ... FOR UPDATE or row versioning, not both. Try it and see. Open three MySQL sessions (A), (B) and (C) to the same database. In (C) issue:

CREATE TABLE test(
    id integer PRIMARY KEY,
    data varchar(255) not null,
    version integer not null
);
INSERT INTO test(id,data,version) VALUES (1,'fred',0);
BEGIN;
LOCK TABLES test WRITE;
In both (A) and (B) issue an UPDATE that tests and sets the row version, changing the winner text in 
each so you can see which session is which:
-- In (A):
BEGIN;
UPDATE test SET data = 'winnerA', 
			version = version + 1
WHERE id = 1 AND version = 0;
-- in (B):
BEGIN;
UPDATE test SET data = 'winnerB',
            version = version + 1
WHERE id = 1 AND version = 0;

Now in (C), UNLOCK TABLES; to release the lock.
(A) and (B) will race for the row lock. One of them will win and get the lock. The other will block on the lock. The winner who got the lock will proceed to change the row. Assuming (A) is the winner, you can now see the changed row (still uncommitted so not visible to other transactions) with a SELECT * FROM test WHERE id = 1.
Now COMMIT in the winner (B) will get the lock and proceed with the update. However, the version no longer matches, so it will change no rows, as reported by the row count result. Only one UPDATE had any effect, and the client application can clearly see which UPDATE succeeded and which failed. No further locking is necessary.

See session logs at pastebin here. I used mysql --prompt="A> " etc to make it easy to tell the difference between sessions. I copied and pasted the output interleaved in time sequence, so it's not totally raw output and it's possible I could've made errors copying and pasting it. Test it yourself to see.

If you had not added a row version field, then you would need to SELECT ... FOR UPDATE to be able to reliably ensure ordering.

If you think about it, a SELECT ... FOR UPDATE is completely redundant if you're immediately doing an UPDATE without re-using data from the SELECT, or if you're using row versioning. The UPDATE will take a lock anyway. If someone else updates the row between your read and subsequent write, your version won't match anymore so your update will fail. That's how optimistic locking works.

The purpose of SELECT ... FOR UPDATE is:

 To manage lock ordering to avoid deadlocks; and

 To extend the span of a row lock for when you want to read data from a row, change it in the application, and write a new row that's based on the original one without having to  use SERIALIZABLE isolation or row versioning.

You do not need to use both optimistic locking (row versioning) and SELECT ... FOR UPDATE. Use one or the other.

References:
http://dba.stackexchange.com/questions/28879/how-to-correctly-implement-optimistic-locking-in-mysql