In logOS a directory is an inode with type FT_DIR. All the data a directory can hold are just directory entries, which are simply other inodes and their names. A directory entry is the struct dirent.

## dirent
```C
/*
 * Directory entry structure
 * Maps filename to inode number
 */
struct dirent {
    uint32_t inode;            /* Inode number (DIRENT_FREE = entry is free) */
    char     name[MAX_FILENAME]; /* Filename (null-terminated) */
};
```
Each dirent is 32 bytes in size, which means that 16 of them can fit into one block.
If the inode variable is set to the sentinal value DIRENT_FREE it signifies a free directory entry. 


## iterating through a directory
almost all the function in this file iterate through directory entries so rather than explain it every time, I am just doing it once over here.

To iterate through all of a directories entries we need to iterate through all of its direct blocks, and for each of the blocks we need to iterate through all the directory entries within it
```C
/* Iterate through all directory blocks */
for (int b = 0; b < DIRECT_BLOCKS; b++) {
	//read block into local_buf
	block_read(dir.blocks[b], local_buf); 
	//convert block to array of dirent pointers so we can read from it
	struct dirent *de = (struct dirent *)local_buf; 
	//iterate through all the entries in the array
	for (uint32_t i = 0; i < DIRENTS_PER_BLOCK; i++) {
		de[i] //this is the current dirent
	}
}
```
I have left out some error checking that is usually done for the sake of clarity

## Free directory entry
A free directory entry signifies that there is nothing in that entry. It means that it can be overwritten/allocated without any issue.
To check whether a direct is free or not we need to check if its inode value is equal to DIRENT_FREE or not
```C
//de is a dirent
if(de.inode != DIRENT_FREE){
	//this is not a free directory entry
}

if(de.inode == DIRENT_FREE){
	//this is a free directory entry
}
```

## dir_list
```C 
int dir_list(uint32_t dir_ino, struct dirent *entries, uint32_t max_entries, uint32_t *count)
```
This function returns the entries of a specified directory, it will either return all or up to the maximum number of entries specified.

return values:
FS_OK: if everything worked successfully
FS_ERR_IO: if inode read or block read fail
FS_ERR_NOT_DIR: if the inode specified is not a directory

Params:
- dir_ino: inode number of the directory we want to search
- \*entries: array of pointers to directory entries
- max_entries: maximum number of entries to be returned
- \*count: 

The function reads the directory inode, checks if it is a directory and then [[#iterating through a directory|iterates through the directory]] and for each [[#Free directory entry|non free entry]] it adds it to the entries array and increments the count


## dir_is_empty
```C
static int dir_is_empty(uint32_t dir_ino) 
```
This function returns 1 if the directory is empty and 0 if it is not.

return values:
1: if the directory is empty
0: if the directory is not empty or if there was an error reading the inode or block

params:
dir_ino: directory inode number

This function reads the directory inode, then [[#iterating through a directory|iterates through the directory]] and for each entry checks if it is either empty or "."(link to same directory) or ".."(link to parent directory)


## dir_lookup
```C
int dir_lookup(uint32_t dir_ino, const char *name, uint32_t *found_ino) 
```
This function searches for a directory entry with the specified name, if found it returns its inode number in found_ino

return values:
FS_OK: if everything was a success
FS_ERR_IO; if there was an error reading the inode or the blocks
FS_ERR_NOT_DIR: if given inode was not a directory
FS_ERR_NOT_FOUND: if there was no directory entry with a matching name

params:
dir_ino: directory inode number
\*name: name of the directory entry we are looking for
\*found_ino: the value of the found inode will be returned in this

The function reads the directory inode, checks if it is a directory and then [[#iterating through a directory|iterates through the directory]] and for each [[#Free directory entry|non free entry]] it compares the name of the entry to the given name, if they are the same returns the inode number of the entry


## dir_add
```C
int dir_add(uint32_t dir_ino, const char *name, uint32_t ino) 
```
this function adds an entry into the given parent directory

return values:
- return value of final [[inode#inode_write|inode_write()]] function call
- FS_ERR_IO: if any of the inode and block read/writes fail
- FS_ERR_NOT_DIR: if the directory inode is not an inode
- FS_ERR_EXISTS: if the name already exists in one of the directory entries
- FS_ERR_INVALID: if name provided is too long
- FS_ERR_NO_SPACE: if there is no space for the new directory

params:
dir_ino: inode number of parent directory
\*name: name of entry to add
ino: inode number of entry to add

This function reads the parent directory inode, checks if it is a directory, checks if the directory to add already exists using [[#dir_lookup]], checks that name given is not too long.
It then [[#iterating through a directory|iterates through the directory]] looking for a free directory entry, and then sets the inode number and name of said free entry to the ones provided.
If no free entry is found a new block is allocated and the first entry in that block is used to set the directory.
If a new directory entry is added the directory size is also increased accordingly

## dir_remove
```C
int dir_remove(uint32_t dir_ino, const char *name)
```
this function removes a directory entry with a matching name from a given directory

return values:
- return value of final [[inode#inode_write|inode_write()]] function call
- FS_ERR_IO: if any of the inode and block read/writes fail
- FS_ERR_NOT_DIR: if the provided directory is not a directory
- FS_ERR_NOT_FOUND: if the directory entry can not be found

params:
dir_ino: inode number of directory we want to remove from
\*name: name of entry to remove

The function reads the directory inode, checks if it is a directory and then [[#iterating through a directory|iterates through the directory]] and for each [[#Free directory entry|non free entry]] it compares the name of the entry to the given name, if they are the same it clears the name and sets the inode to free, it also decreases the size of the directory accordingly.

## fs_mkdir
```C
int fs_mkdir(uint32_t parent_ino, const char *name) 
```
This function creates a new directory and adds it into the given parent directory

return values:
- returns the inode number of the newly created directory on success
- FS_ERR_INVALID: if the name of directory to add is "." or ".."
- FS_ERR_IO: if the inode_write function call fails
- errors from any internal function calls

params:
- parent_ino: inode number of parent directory
- \*name: name of directory to create

this function first checks that the name of directory is not "." or "..", after that it [[inode#inode_alloc|allocates the inode]] initialises it by zeroing out the memory, setting the size as 0 and setting the type as FT_DIR, then it adds the current directory as "." and parent directory as ".." and finally adds the newly created directory into the parent directory before returning the inode number of the directory.


## fs_rmdir
```C
int fs_rmdir(uint32_t parent_ino, const char *name) 
```
This function removes a directory from a given parent directory

return values:
- return value of final [[inode#inode_free|inode_free()]] call
- FS_ERR_INVALID: if the name of the directory to remove is "." or ".."
- FS_ERR_IO: if the inode_read function call fails
- FS_ERR_NOT_DIR: if directory to remove isn't a directory
- FS_ERR_NOT_EMPTY: if directory to remove isn't empty
- return values of other failed internal function calls

params:
- parent_ino: inode number of parent directory
- \*name: name of directory to remove

This function first checks that the name of the directory is not "." or "..", after that it looks up the directory in the parent directory, checks if it is actually a directory, checks if it is empty, removes "." and ".." from the directory, removes the directory from the parent directory and finally frees the inode