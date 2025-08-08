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
./hash-table-tester -t 4 -s 25000
```

Version 1 is slower than the base version due to the overhead of locking the entire table for every operation. The single lock creates a bottleneck when multiple threads try to access the table simultaneously.

## Second Implementation
In the `hash_table_v2_add_entry` function, I implemented fine-grained locking where each bucket has its own pthread_mutex_t bucket_lock. This allows multiple threads to access different buckets concurrently, improving performance under high contention.

### Performance
```shell
./hash-table-tester -t 4 -s 25000
```

Version 2 shows significant improvement over version 1, especially with more threads. The bucket-level locking reduces contention and allows for better parallelization. However, there's still some overhead from managing multiple mutexes.

## Analysis

The fine-grained locking approach in v2 provides better scalability because:
- Multiple threads can access different buckets simultaneously
- Reduced contention compared to the single lock in v1
- Better cache locality for frequently accessed buckets

The trade-off is increased memory usage (one mutex per bucket) and slightly more complex code, but the performance benefits outweigh these costs for concurrent workloads.

## Cleaning up
```shell
make clean
```