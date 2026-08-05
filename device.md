In logisim a device is a pseudo file, that like all other files can be opened, closed, read from and written to, however unlike a file all of these operations are defined in the device driver and each device has their own implementation of each function.

The device.c file can be thought of as an abstract device, it can call the read/write/open/close functions of any device.

Each device can be uniquely identified by the combination of its major number and minor number. The major number specifies what type of device it is, and the minor number identifies which specific device of said type it is.

## device_ops struct
```C
struct device_ops {
    int (*open)(uint8_t minor);
    int (*close)(uint8_t minor);
    int (*read)(uint8_t minor, void *buf, uint32_t len);
    int (*write)(uint8_t minor, const void *buf, uint32_t len);
};
```
This struct consists of functions pointers to open,close,read and write functions of a device thus it represents all the functions of a device. Each type of device will have its own device_ops struct, with its own functions added in.

e.g.
```C
static struct device_ops console_ops = {
    .open  = 0,
    .close = 0,
    .read  = console_read,
    .write = console_write
};
static struct device_ops display_ops = {
    .open = 0,
    .close = 0,
    .read = 0,
    .write = display_write
};
```
All device do not need to have all the functions

## device_table
```C
static struct device_ops *device_table[MAX_DEVICES];
```
This array holds all the registered devices, indexed by their major number

## device_register
```C
int device_register(uint8_t major, struct device_ops *ops)
```
Adds a device into the device_table.

return values:
FS_OK: on success
FS_ERR_INAVLID: if the major number exceeds the maximum number of devices
FS_ERR_EXISTS: if the device has already been registered

params:
major: major number of the device to be added
\*ops: pointer to device_ops struct of the device

This functions checks if the major number is less than the maximum number of allowed devices, checks if the device has already been created and then it adds the device into the device table.

without error checking this function is essentially one line
```C
int device_register(uint8_t major, struct device_ops *ops) {
	device_table[major] = ops;
}
```

## device_unregister
```C
int device_unregister(uint8_t major)
```
removes a device from the device table

return values:
FS_OK: on success
FS_ERR_INVALID: if the major number exceeds the maximum number of devices

params:
major: major number of the device to be removed

This function checks if major number is less than the maximum number of allowed devices, and then zeros the section of memory occupied by this device in the device table

## open, close, read and write
most of these functions are pretty much identical, in the sense that they essentially just call the corresponding function of a given device. Ignoring error checking and some differences they all essentially do this
```C
int device_func1(uint8_t major, uint8_t minor){
	return device_table[major]->func1(minor);
}
```


## device_open
```C
int device_open(uint8_t major, uint8_t minor) 
```
This function calls the given device's open function, if it has one

return values:
- return value of open() function if it exists
- FS_OK: as having no open function is alright
- FS_ERR_NOT_FOUND: if invalid major number or device doesn't exist

params:
- major: major number of device
- minor: minor number of device

This function checks if the major number is valid and if the device exists. It then checks if the device has an open function, if it does it calls the open function, if it does not exist it return ok, as not having an open handler is fine.


## device_close
```C
int device_close(uint8_t major, uint8_t minor) 
```
This function calls the given device's close function, if it has one

return values:
- return value of close() function if it exists
- FS_OK: as having no close function is alright
- FS_ERR_NOT_FOUND: if invalid major number or device doesn't exist

params:
- major: major number of device
- minor: minor number of device

This function checks if the major number is valid and if the device exists. It then checks if the device has a close function, if it does it calls the close function, if it does not exist it return ok, as not having an close handler is fine.



## device_read
```C
int device_read(uint8_t major, uint8_t minor, void *buf, uint32_t len) 
```
This function calls the given device's read function

return values:
- return value of device's read function
- FS_ERR_NOT_FOUND: if invalid major number or device doesn't exit
- FS_ERR_INVALID: if device doesn't have a read function

params:
- major: major number of device
- minor: minor number of device
- \*buf: buffer to be read into
- len: length to be read

This function checks if the major number is valid and if the device exists. It then checks if the device has a read function, if it does, it calls the read function.


## device_write
```C
int device_write(uint8_t major, uint8_t minor, void *buf, uint32_t len) 
```
This function calls the given device's write function

return values:
- return value of device's write function
- FS_ERR_NOT_FOUND: if invalid major number or device doesn't exit
- FS_ERR_INVALID: if device doesn't have a write function

params:
- major: major number of device
- minor: minor number of device
- \*buf: buffer to be written from
- len: length of buffer 

This function checks if the major number is valid and if the device exists. It then checks if the device has a write function, if it does, it calls the write function.