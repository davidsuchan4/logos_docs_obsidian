
## block_init
``` C
int block_init(void)
```
essentially just a wrapper around [[logos_parition.c#logos_partition_init | logos_partition_init()]] 

## block_read
```C
int block_read(uint32_t block_num, void *buf)
```
essentially wrapper around [[logos_parition.c#logos_partition_read|logos_partition_read()]]
does check if blocks have been initialised, if pointer to buffer is null and if the block to be read is within the bounds of the partition before calling [[logos_parition.c#logos_partition_read|logos_partition_read()]] 


