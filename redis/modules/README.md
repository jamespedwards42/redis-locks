# Locks API reference

## LOCKS.MUTEX
The intended usage of this lock is to serve leader elections amongst distributed services.  Be aware that after a failover of your Redis server a new service could claim leadership before your previous leader has realized it has lost leadership.  To protect against this corner case, have your service acquire the lock twice, effectively waiting 2 full checkin periods, when the previous owner is null.  This will ensure that the previous leader has attempted to refresh its claim and discovered it is no longer the leader.

### `LOCKS.MUTEX.TRY.ACQUIRE lockName ownerId pexpire`
------
Tries to acquire the lock if it is not currently owned by anyone else.  If the lock is newly acquired or `ownerId` matches the current owner the expiration will be set to `pexpire` milliseconds.

#### Return value
[Array reply](http://redis.io/topics/protocol#array-reply):  The previous owner, the current owner and the time to live in milliseconds.

####Example
redis> LOCKS.MUTEX.TRY.ACQUIRE myLock myId 1000

1. null
2. "myId"
3. 1000

redis> LOCKS.MUTEX.TRY.ACQUIRE myLock myId 1000

1. "myId"
2. "myId"
3. 1000

redis> LOCKS.MUTEX.TRY.ACQUIRE myLock someOtherId 1000

1. "myId"
2. "myId"
3. 999

### `LOCKS.MUTEX.TRY.RELEASE lockName ownerId`
------
Frees the lock if it is currently held by `ownerId`.

#### Return value
[Bulk string reply](http://redis.io/topics/protocol#bulk-string-reply):  The owner at the time of this call.

####Example
redis> LOCKS.MUTEX.TRY.TRY_RELEASE myLock myId

null

redis> LOCKS.MUTEX.TRY.ACQUIRE myLock myId 2000

1. null
2. "myId"
3. 2000

redis> LOCKS.MUTEX.TRY.TRY_RELEASE myLock myId

"myId"