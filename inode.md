An inode a.k.a an index node is a data structure which stores the meta data for a file or directory, it stores everything other than the name and the actual data.o

Inodes are stored in an inode table, these are the blocks of memory after the bitmap. 

The [[superblock]] stores the following information about inodes:
- total number of inodes
- number of free inodes
- starting block of inode table
- length of inode table in blocks

## root inode
the root inode is the inode for the root directory. It has the inode number of 0. It is created in the fs_format() function

## inode struct

```C
/*
 * Inode structure
 * Contains metadata about a file or directory
 * For device files: major/minor store device numbers, blocks[] unused
 */
struct inode {
    uint32_t size;             /* File size in bytes */
    uint8_t  type;             /* File type (FT_FREE, FT_FILE, FT_DIR, FT_CHARDEV) */
    uint8_t  major;            /* Major device number (for FT_CHARDEV) */
    uint8_t  minor;            /* Minor device number (for FT_CHARDEV) */
    uint8_t  link_count;       /* Number of hard links to this inode */
    uint16_t blocks[DIRECT_BLOCKS];  /* Direct block pointers (16-bit indices) */
};

```


## inode_read
```C
int inode_read(uint32_t ino, struct inode *ip) 
```
function reads a specified inode from memory into the given inode
the function returns 
- FS_OK on success
- FS_ERR_INVALID if inode number is greater than total inodes
- FS_ERR_IO if reading the block fails
 
ino is inode number
\*ip is pointer to inode that will be read into

the function calculates where the inode is 
- the block its in
- the offset within that block
then reads the block and copies the inode from that block into the inode pointed to by ip

## inode_write
```C
int inode_write(uint32_t ino, const struct inode *ip) 
```
function write the given inode into the inode specified in memory
the function returns 
- return value of final [[Blocks#block_write|block_write()]] function call
- FS_ERR_INVALID if inode number is greater than total inodes
- FS_ERR_IO if reading the block fails

ino is inode number
\*ip is pointer to inode that will be read into

the function calculates where the inode is 
- the block its in
- the offset within that block
then reads the block and copies the  inode pointed to by ip into the inode from that block 
it works the exact same as inode_read, except where its being copied from and written to a swapped

## inode_alloc
```C
int inode_alloc(void)
```
this function allocates an inode and returns the inode number of the allocated inode
the function returns
- inode number on success
- FS_ERR_NO_SPACE if no free inodes remain
- FS_ERR_IO if any of the inode read/write or block writes fail

the function iterates through all the inodes until it finds a free inode
once it finds a free inode it 
- clears the inode (sets that memory segment to all 0s)
- sets the link_count to 1
- decrements the number of free inodes
- returns the inode number

## inode_free
```C
int inode_free(uint32_t ino) 
```
This function frees the specified inodes and any data blocks associated with it
the funciton returns
- return value of final [[Blocks#block_write|block_write()]] function call
- FS_ERR_IO if any of the inode read/writes fail

the reads the specified inode
iterates through all of its direct blocks and frees all of them
clears the inode (sets the memory segment to all 0s)
sets the inode type to FT_FREE
increments the number of free inodes


## inode_get_type, inode_get_size, inode_get_device
```C
int inode_get_type(uint32_t ino, uint8_t *type) 
int inode_get_size(uint32_t ino, uint32_t *size) 
int inode_get_device(uint32_t ino, uint8_t *major, uint8_t *minor) 
```
all of these functions work in pretty much the same way
they read the specified inode and then set the information in the specified return arguments.
the functions return 
- FS_OK on success
- FS_ERR_IO if the [[inode#inode_read|inode_read()]] funciton fails 