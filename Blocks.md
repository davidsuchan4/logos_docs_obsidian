
## block_init
``` C
int block_init(void)
```
essentially just a wrapper around [[logos_parition.c#logos_partition_init | logos_partition_init()]] 

## block_read
```C
int block_read(uint32_t block_num, void *buf)
```
reads specified block into given buffer
returns FS_ERR_IO on error and FS_OK on success

essentially wrapper around [[logos_parition.c#logos_partition_read|logos_partition_read()]], the length to be read is the size of a block 
does check if blocks have been initialised, if pointer to buffer is null and if the block to be read is within the bounds of the partition before calling [[logos_parition.c#logos_partition_read|logos_partition_read()]] 


## block_write
```C
int block_write(uint32_t block_num, const void *buf)
```
writes buffer to the specified block