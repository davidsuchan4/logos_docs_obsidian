
## console_read

paramaters:
 - minor - identifies the specific instance of the device being used
 - buf - where we will output whataver we read
 - len - the length of the data we want to read

`void(minor)`minor is unused on the esp32 since there is only one console we dont have to differentiate between which console we are using so we cast it to void

`uint8_t *dst = (uint8_t *)buf;` since we pass buf as a void pointer we do not know what kind of data it points to, as such we cannot safely index or dereference it. we first need to cast it to a pointer to a byte after which we can safely work with it

```
uint32_t i = 0;
while (i < len) {
	dst[i++] = (uint8_t)logos_getchar();
}
```
the loop iterates from 0 to the size of len - 1. in every iteration it reads a character from the console and writes it to the memory that dst is pointing to using [[logos_getchar()]] after which we move dst to the next byte 

`return i` the length of the data read is then returned


## console_write

paramaters:
- minor - identifies the specific instance of the device being used
- buf - a pointer to the start of the data to be written to the console
- len - the length of the data to be written to the console

`return logos_write(buf, (int)len);` - this function is simply a wrapper around [[logos_write()]] in which we pass the buf pointer and the size of the data through len 


## console_dev_init

this function exposes the console device to the kernel allowing it to be used for system calls through [[device_register()]] by passing the constant CONSOLE_MAJOR which is the major number for the device as a whole and the console_ops struct which defines the functions that can be performed on this device





