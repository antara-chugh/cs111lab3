# Hash Hash Hash
TODO introduction

## Building
```shell
TODO
```

## Running
```shell
TODO how to run and results
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added "pthread_mutex_lock(&hash_table->lock)" to create a lock at the beginning of the function. With this, I made the entire a function an atomic operation thus preventing multiple threads from adding entries and updating the hash tables at the same time, eliminating data races. The locks are then unlocked before the function returns (in the case that the value already exists in the table) or after the new key and value is added. In addition, I added checks to ensure that the lock and unlock functions ran correctly with no error code. If these functions returned a non zero exit code, I called the destroy table function to free the hash table memory.

To support all of this, I added a lock to the hash_table_v1 struct, intialized the lock in the hash_table_v1 create function, and destroyed the lock in the hash_table_v1_destroy function.


### Performance
```shell
TODO how to run and results
```
Version 1 is a little slower than the base version because the during the add entry function, the hash table has been locked, so only one thread can update the hash table at a time and must perform the entire update before another thread can update the table. Moreover, there is minor overhead from intializing the lock in the create function, creating it add entry, and then destroying it in the destory function. In the base version, threads update (add entries, delete etc) the hash table in parallel, which effects correctness but has faster performance. Moreover, there is no overhead from creating locks. 

## Second Implementation
In the `hash_table_v2_add_entry` function, I add a lock to the bin the thread is trying to access via "pthread_mutex_lock(&hash_table_entry->lock)". This way, other threads can access other bins at the same time. I then unlock the lock when the function returns or ends. Additionally, I add the same checks to ensure that the lock and unlock functions exit properly, calling hash_table_v2_destroy if not and exiting with an error code. 

To support all of this, I added a lock to the hash_table_entry struct, intialized the lock for each bin in the for loop intializing entries in the hash_table_v2_create function, and destroyed the every bin's lock in the hash_table_v1_destroy function.



### Performance
```shell
TODO how to run and results
```
V2 runs faster than both V1 and the base implementation

In V1, locking the entire entry function causes the entire shared hash table to be locked. Even if other threads are trying to update different bins, there access is blocked until a thread is done adding an entry to the hashtable.
In V2, we only lock the bin that is being updated. This way, if threads are accessing different bins (meaning there are no collisions) they can do so, enabling more parallelization and having the code run faster. 

TODO more results, speedup measurement, and analysis on v2

## Cleaning up
```shell
TODO how to clean
```