In logisim a device is a pseudo file, that like all other files can be opened, closed, read from and written to, however unlike a file all of these operations are defined in the device driver and each device has their own implementation of each function.

The device.c file can be thought of as an abstract device, it can call the read/write/open/close functions of any device.

Each device has a major number and a minor number. The major number specifies what type of device it is, and the minor number identifies which specific device of said type it is.

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
