# Hash Hash Hash
In this lab, we explored the ways to implement a hash table safe to use with multithreading and concurrency. Two possible implementations are provided, one where only a single lock is used and one where multiple locks are used.  

## Building
To build, run make. 

## Running
To run, run 
./hash-table-tester -t (#number of threads to use, by default 4) -s (#number of entries to add, by default 25000)

Example:
./hash-table-tester -t 8 -s 500000

This will run the base implementation and then the two implementations (v1 and v2) with the specified number of threads, and outputs number of μs for each implemeentation, as well as the number of missing entries in the hash table. Correct implementations have no missing entries.

## First Implementation
In the `hash_table_v1_add_entry` function, I added "pthread_mutex_lock(&hash_table->lock)" to create a lock at the beginning of the function. With this, I made the entire a function an atomic operation thus preventing multiple threads from adding entries and updating the hash tables at the same time, eliminating data races. The locks are then unlocked before the function returns (in the case that the value already exists in the table) or after the new key and value is added. In addition, I added checks to ensure that the lock and unlock functions ran correctly with no error code. If these functions returned a non zero exit code, I called the destroy table function to free the hash table memory.


To support all of this, I added a lock to the hash_table_v1 struct, intialized the lock in the hash_table_v1 create function, and destroyed the lock in the hash_table_v1_destroy function. Each pthread_mutex* call also has proper error handling with memory management.


### Performance
After running ./hash-table-tester with the desired parameters 

Version 1 is slower than the base version because the during the add entry function, the hash table has been locked, so only one thread can update the hash table at a time and must perform the entire update before another thread can update the table. Moreover, there is  overhead from intializing the lock in the create function, creating it add entry, and then destroying it in the destory function. In essence, this causes the code to run in a sequential manner, and is essentially comparable to the serial base solution but with locak overhead. 

## Second Implementation
In the `hash_table_v2_add_entry` function, I added a lock to the bin the thread is trying to access via "pthread_mutex_lock(&hash_table_entry->lock)". This way, other threads can access other bins at the same time. I then unlock the lock when the function returns or ends. Additionally, I add the same checks to ensure that the lock and unlock functions exit properly, calling hash_table_v2_destroy if not and exiting with an error code. 

To support all of this, I added a lock to the hash_table_entry struct, intialized the lock for each bin in the for loop intializing entries in the hash_table_v2_create function, and destroyed the every bin's lock in the hash_table_v1_destroy function. Each pthread_mutex* call also has proper error handling with memory management.



### Performance
After running ./hash-table-tester with the desired parameters, the "Hash Table v2" line in the output will specify the number of μs the implemeentation took and the number of missed entries.

V2 runs faster than both V1 and the base implementation.

In V1, locking the entire entry function causes the entire shared hash table to be locked. Even if other threads are trying to update different bins, there access is blocked until a thread is done adding an entry to the hashtable.
In V2, we only lock the bin that is being updated. This way, if threads are accessing different bins (meaning there are no collisions) they can do so, enabling more parallelization and having the code run faster. This thus is also faster than the base implementation, as threads are able to do different operations at the same time, which is faster than a serial implementation. 





## Cleaning up
run "make clean" in the shell to clean the .o files