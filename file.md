In logos a file is an inode with the FT_FILE type. 

## file_create
```C
int file_create(uint32_t dir_ino, const char *name) 
```
This function creates a file with the given name in the specified directory

return values:
- returns the inode number of the newly created file
- FS_ERR_INVALID: if the name of the file to be created is "." or ".."
- FS_ERR_IO: if the [[inode#inode_write|inode_write()]] function fails
- errors from any internal function calls

params:
- dir_ino: inode number of directory within which the file will be created
- \*name: name of file to be created

This function first checks that the name of the file to create is not "." or "..", then it creates a new inode, initialises it by zeroing the memory, setting the inode type to FT_FILE and setting the size to 0. Finally it adds the newly created file to the given directory before returning the inode of the file.


## file_read
```C
int file_read(uint32_t ino, uint32_t offset, void *buf, uint32_t len) 
```
This function reads len number of bytes from a certain offset in the file into buf.

return values:
