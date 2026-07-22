
## logos_putchar

this function simply takes in a character and uses the built in [[esp_rom_output_putc()]] function to write a character to the console

## logos_getchar

this function simply reads a character from the console

`uint8_t ch` - first a variable is created into which the character will be stored

```
while (esp_rom_output_rx_one_char(&ch) != 0) {
	;
}
```

the built-in function [[esp_rom_output_rx_one_char()]] to polls the console until a character is available to read

`return ch` once a character is read and written into ch - it is returned

## logos_write

#### paramaters:
 - `const void *buf` - a void pointer pointing to the start of memory from which the data should be written to the console
 - `int len` - the length of the data to be written

`const unsigned char *s = buf;` first the void pointer is recasted to a char pointer so that it can be read from

```
if (buf == NULL || len < 0) {
	return -1;
}
```

an error check is performed to make sure there is data to write

```
for (int i = 0; i < len; ++i) {
	logos_putchar((char)s[i]);
}
```
the loop iterates through the whole buffer writing every character to the console using [[logos_putchar()]]

`return len` - the length of the data written is returned to the caller


##  logos_printf

#### paramaters:
 - `const char *fmt` a char pointer to the beginning of the format string that the caller passes
 - `...` - the variables to fill the placeholders in the format string - they need to match the placeholder in the exact same order

```
va_list args;
int written;
```
a [[va_list]] is defined to hold the traversal state needed to find the variable arguments passed as the second paramater 
written is defined as a variable to hold the length of the message written

[[va_start()]] is used to initialise args to be able to find the first variable argument after which it is able to find the rest

[[esp_rom_printf()]] is a builtin in function that prints formatted strings to the esp32 console device

[[va_end()]] cleans up the initialised va_list

`return written` the length of the data written to the console is returned

