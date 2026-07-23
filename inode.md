An inode a.k.a an index node is a data structure which stores the meta data for a file or directory, it stores everything other than the name and the actual data.

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