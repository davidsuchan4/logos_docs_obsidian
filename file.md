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
This function reads len number of bytes from a certain offset in the file into buf, and returns the number of bytes successfully read

return values:
- returns the number of bytes successfully read 
- FS_ERR_IO: if the inode or block reads fail
- FS_ERR_IS_DIR: if the given file is actually a directory
- 0: if the offset to read from is out of bounds

params
- ino: inode number of file to read from
- offset: number of bytes after which we want to start reading
- \*buf: buffer into which the contents of file will be read
- len: number of bytes to read

This function reads the inode, checks if it is a device file, if it is it calls the [[device#device_read|device_read()]] function. Makes sure the file isn't a directory. Adjusts the length to make sure we are not reading past the end of the file. It then iterates through the relevant blocks of the file and reads from them and writes their content into the buffer. It also keeps track of how many bytes have been read and returns that.