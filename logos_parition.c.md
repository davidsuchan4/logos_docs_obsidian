
## logos_partition_init

```
if (esp_flash_default_chip != NULL &&
	esp_flash_chip_driver_initialized(esp_flash_default_chip)) {
	return 0;
}
```
the first check makes sure a default flash-chip structure exists. if it didnt then the pointer wouldnt be safe to pass to another function since it wouldnt be pointing to anything

the function it is passed to checks if the flash chip is is already initialised and if it is then it simply exits with a success code

```
err = esp_flash_init_default_chip();
if (err != ESP_OK) {
	logos_printf("logos_partition: esp_flash_init_default_chip failed: 0x%08lx\n",
						  (unsigned long)err);
	return -1;
}
```
 if it is not already initialised then the function attempts to initialise it printing an error message on failure

## logos_partition_read

```C
int logos_partition_read(uint32_t offset, void *buf, uint32_t len)
```
essentially wrapper around **esp_flash_read()** which is a built in function
does check if the block to be read is within range and that pointer to buffer is not null before calling

**esp_flash_read()** built in function to read data from flash memory

## logos_partition_write
```C
int logos_partition_write(uint32_t offset, const void *buf, uint32_t len)
```
essentially a wrapper around **esp_flash_write()**

**esp_flash_write()** built in function to write data to flash memory, this function itself does not have any alignment requirements however it does need the region being written to, to be erased. The erase function, **esp_flash_erase_region()** can only erase sectors i.e. 4096 bytes at a time


## logos_partition_erase
```C
int logos_partition_erase(uint32_t offset, uint32_t len)
```
essentially a wrapper around **esp_flash_erase_region()**
which means that it can only erase sectors i.e. 4096 bytes at a time