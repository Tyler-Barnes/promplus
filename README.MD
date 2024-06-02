## **PROMPLUS Library** for Arduino

**README Written by:** _Christopher Andrews_.

**Modified by:** _Tyler Barnes_. 

### **What is the PROMPLUS library?**

The PROMPLUS.h library is a fork of the standard EEPROM.h library bundled in the Arduino core. The core library has been modified to perform redundant writes and reads, making sure that accessing the EEPROM always results in valid data, even if unexpected power loss has corrupted a location within EEPROM.   

It is recommended to always enable Brown Out Detection (BOD) when you are working with the EEPROM on your AVR project. Enabling BOD on AVR chips can help ensure EEPROM does not get corrupted. While it is very effective, it is not fullproof on it's own. 

Using this library in combination with BOD will yield rock solid EEPROM access in your next project. 

The interface is the exact same as the core library, all while utilizing redundant writes to ensure data validity. This library will work on many AVR devices like ATtiny and ATmega chips.

### **How to install it**
The PROMPLUS.h library can be downloaded from the releases section of this repository. 

Once the .zip file has been downloaded, click on the `Sketch` tab within the Arduino IDE, click on `Include Library`, then click on `Add .ZIP Library...`. 

The resulting dialog will allow you to locate the library within your computer's directories, and open it. 


### **How to use it**
To add its functionality to your sketch you'll need to reference the library header file. You do this by adding an include directive to the top of your sketch.

```c++
#include <PROMPLUS.h>

void setup(){

}

void loop(){

}

```

The library provides a global variable named `PROM`, you use this variable to access the library functions. The methods provided in the EEPROM class are listed below.

**By including the PROMPLUS.h library, you will automatically be including the standard EEPROM.h library as well, allowing access to the unmodified functionality of the original.**

### **Library functions**

#### **`PROM.read( address )`** 

This function allows you to read a single byte of data from the eeprom.
Its only parameter is an `int` which should be set to the address you wish to read.

The function returns an `unsigned char` containing the value read.

#### **`PROM.write( address, value )`** 

The `write()` method allows you to write a single byte of data to the EEPROM.
Two parameters are needed. The first is an `int` containing the address that is to be written, and the second is a the data to be written (`unsigned char`).

This function does not return any value.

#### **`PROM.update( address, value )`** 

This function is similar to `PROM.write()` however this method will only write data if the cell contents pointed to by `address` is different to `value`. This method can help prevent unnecessary wear on the EEPROM cells.

This function does not return any value.

### **An important note on how indexing works**

In the standard EEPROM.h library, the address you pass into the methods will be the actual address accessed wthin the EEPROM. 

In this modified library, the address that you pass in will be multiplied by the number of redundancies that it is designed to write/read. 

In PROMPLUS.h: 
- The address `0` is equal to `0x00` in EEPROM.
- The address `1` is equal to `0x03` in EEPROM.
- etc...

If for some reason you want to increase the number of redundant writes that the library performs, you can do so. Please see the note at the bottom of this file regarding `PROM.duplicates`.  

#### **`PROM.get( address, object )`** 

This function will retrieve any object from the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to read.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for convenience.

#### **`PROM.put( address, object )`** 

This function will write any object to the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to write.

This function uses the _update_ method to write its data, and therefore only rewrites changed cells.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for convenience.

### **An important note on the `PROM.put()` and `PROM.get()` methods**

In the standard EEPROM.h library, the address you pass in is the actual address within the EEPROM. This in my opinion can lead to issues if you are expecting the indexing to act like an array and respect the width of the given datatype. 

I have modified the addressing behavior to index based on the `sizeof()` the datatype you pass in. 

I did this so that the indexing behaves just like indexing into an array of a given type. This means that you will not have to keep track of the `sizeof()` your datatypes when you want to iterate through the data as if it were an array. 

However, just like an array, you will need to keep the datatype consistant throughout to keep the index aligned.   

You can expect these two examples to index the exact same way. 
```c++
uint16_t array[5]; 

uint16_t number = 1000; 

for (int i = 0; i < 5; i++) {
  array[i] = number * i; 
}
```
```c++

uint16_t number = 1000; 

for (int i = 0; i < 5; i++) {
  PROM.put(i, number * i);
}
```

The resulting data for either of the above examples will equal:
```c++
{0, 1000, 2000, 3000, 4000}
```

In this example: 
- The index of `0` points to the address `0x00` in the EEPROM. 
- The index of `1` points to the address `0x06` in the EEPROM. 

This is because each write will write three times for redundancy, and the datatype being written is 2 bytes wide. 

`2 * 3 = 6.`

The memory dump of the `PROM.put()` example looks like: 
```bash
0x0000: 00,00,00,00,00,00,E8,E8,E8,03,03,03,D0,D0,D0,07
0x0010: 07,07,B8,B8,B8,0B,0B,0B,A0,A0,A0,0F,0F,0F,FF,FF
0x0020: FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF
```

The only other thing I would add to the above example is a base address since you may not always want to start indexing from address `0x00` in the EEPROM. 

```c++

uint16_t number = 1000; 
uint16_t baseAddress = 0x10; // base address can be anything you want
for (int i = 0; i < 5; i++) {
  PROM.put(baseAddress + i, number * i);
}
```

```bash
0x0000: FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF
0x0010: 00,00,00,00,00,00,E8,E8,E8,03,03,03,D0,D0,D0,07
0x0020: 07,07,B8,B8,B8,0B,0B,0B,A0,A0,A0,0F,0F,0F,FF,FF
0x0030: FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF,FF
```

#### **Subscript operator: `PROM[address]`** 

This operator allows using the identifier `PROM` like an array.  
EEPROM cells can be read _and_ **_written_** directly using this method.

The only valid datatypes for use with this operator are `unsigned char`, or `uint8_t`. 

This operator returns a reference to the EEPROM cell.

```c++
uint8_t val;

//Read first EEPROM cell.
val = PROM[0];

//Write first EEPROM cell.
PROM[0] = val;

//Compare contents
if(val == PROM[0]){
  //Do something...
}
```

#### **`PROM.length()`**

This function returns an `unsigned int` containing the number of cells in the EEPROM. The value returned is divided by the number of duplicate writes that the library utilizes. 

If the total number of byte sizes cells in the EEPROM is 1024, then the PROM.length() call will yield the result 341, since the default number of redundant writes in the library is 3. Note that the value returned is the result of integer math, and any float remainders are truncated. 

---

### **Advanced features**

This library uses a component based approach to provide its functionality. This means you can also use these components to design a customized approach. Two background classes are available for use: `PRef` & `PPtr`.

#### **`PRef` class**

This object references an EEPROM cell.
Its purpose is to mimic a typical byte of RAM, however its storage is the EEPROM.
This class has an overhead of two bytes, similar to storing a pointer to an EEPROM cell.

```C++
PRef ref = PROM[10]; //Create a reference to 11th cell.

ref = 4; //write to EEPROM cell.

unsigned char val = ref; //Read referenced cell.
```

#### **`PPtr` class**

This object is a bidirectional pointer to EEPROM cells represented by `PRef` objects.
Just like a normal pointer type, this type can be dereferenced and repositioned using 
increment/decrement operators.

```C++
PPtr ptr = 10; //Create a pointer to 11th cell.

*ptr = 4; //dereference and write to EEPROM cell.

unsigned char val = *ptr; //dereference and read.

ptr++; //Move to next EEPROM cell.
```

#### **`PROM.begin()`**

This function returns an `PPtr` pointing to the first cell in the EEPROM.  
This is useful for STL objects, custom iteration and C++11 style ranged for loops.

#### **`PROM.end()`**

This function returns an `PPtr` pointing at the location after the last EEPROM cell.  
Used with `begin()` to provide custom iteration.

**Note:** The `PPtr` returned is invalid as it is out of range. In fact the hardware causes wrapping of the address (overflow) and `PROM.end()` actually references the first EEPROM cell.

#### **`PROM.duplicates`**

If for whatever reason you would like to increase the number of redundant writes, then you can change the member variable `PROM.duplicates`. Note that you need at least three for the `PROM.read()` method to properly be able to compare each stored value and determine which value is valid. The default value is `3`. 

```c++
PROM.duplicates = 4; 
```