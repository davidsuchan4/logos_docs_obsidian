
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
to write to a section of flash memory, esp requires that it be erased first and then written to
when erasing a section of flash memory, esp can only erase sectors i.e. 4096 bytes at a time
this is why we need to 
- read the whole sector from flash in
- modify the sector
- erase the sector from flash
- write the modified sector into flash