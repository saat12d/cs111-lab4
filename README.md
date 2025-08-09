# Hash Hash Hash

This lab implements two different thread-safe hash table implementations using pthreads. The first version uses a single table-wide lock, while the second version uses fine-grained bucket-level locking for better concurrency.

## Building
```shell
make
```

## Running
```shell
./hash-table-tester [-t threads] [-s size]
```

Default values:
- threads: 4
- size: 25000 (entries per thread)

Example:
```shell
./hash-table-tester -t 8 -s 10000
```

## First Implementation
In the `hash_table_v1_add_entry` function, I added a single pthread_mutex_t table_lock that protects the entire hash table. This ensures thread safety by serializing all access to the table.

### Performance
```shell
./hash-table-tester -t 4 -s 50000
Generation: 24,904 usec
Hash table base: 66,791 usec
  - 0 missing
Hash table v1: 180,330 usec
  - 0 missing
```

Version 1 is actually slower than the base version, which I found surprising at first. The single lock creates a bottleneck when multiple threads try to access the table simultaneously. It's about 2.7x slower than the base version, which shows how much overhead the global lock adds.

## Second Implementation
In the `hash_table_v2_add_entry` function, I implemented fine-grained locking where each bucket has its own pthread_mutex_t bucket_lock. This allows multiple threads to access different buckets concurrently.

### Performance
```shell
./hash-table-tester -t 4 -s 50000
Generation: 24,904 usec
Hash table base: 66,791 usec
  - 0 missing
Hash table v2: 20,674 usec
  - 0 missing
```

Version 2 turned out much better! It's actually faster than the base version (about 3.2x faster) and way faster than version 1 (8.7x faster). The bucket-level locking really helps reduce contention between threads.

## Analysis

The fine-grained locking approach in v2 works well because:
- Multiple threads can access different buckets at the same time
- Less contention compared to the single lock in v1
- Better cache performance for frequently accessed buckets

The main trade-off is using more memory (one mutex per bucket) and the code is a bit more complex, but the performance improvement is definitely worth it. It's interesting that v2 is faster than the base version - I think this is because the base version doesn't use threads at all, while v2 can take advantage of multiple cores effectively.

## Cleaning up
```shell
make clean
```