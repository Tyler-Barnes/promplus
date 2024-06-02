## **PROMPLUS Library** for Arduino

**README Written by:** _Christopher Andrews_.

**Modified by:** _Tyler Barnes_. 

### **What is the PROMPROM library.**

The PROMPLUS library is a fork of the standard EEPROM library bundled in the Arduino core. the core library has been modified to perform multiple writes and reads 

The PROMPLUS.h library provides an easy to use interface to interact with the internal non-volatile storage found in AVR based Arduino boards, all while utilizing redundant writes to ensure data validity. This library will work on many AVR devices like ATtiny and ATmega chips.

### **How to use it**
The EEPROM library is included in your IDE download. To add its functionality to your sketch you'll need to reference the library header file. You do this by adding an include directive to the top of your sketch.

```Arduino
#include <PROMPLUS.h>

void setup(){

}

void loop(){

}

```

The library provides a global variable named `PROM`, you use this variable to access the library functions. The methods provided in the EEPROM class are listed below.

By including the PROMPLUS library, you will automatically be including the standard EEPROM library as well, allowing access to the unmodified functionality of the original. 

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

#### **`PROM.get( address, object )`** 

This function will retrieve any object from the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to read.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for convenience.

#### **`PROM.put( address, object )`** 

This function will write any object to the EEPROM.
Two parameters are needed to call this function. The first is an `int` containing the address that is to be written, and the second is the object you would like to write.

This function uses the _update_ method to write its data, and therefore only rewrites changed cells.

This function returns a reference to the `object` passed in. It does not need to be used and is only returned for convenience.

#### **An inportant note on the `PROM.put()` and `PROM.get()` methods**

In the standard EEPROM library, the address you pass in is the actual address whithin the EEPROM. This in my opinion can lead to issues if you are expecting the indexing to act like an array and respect the width of the given data type. 

I have modified the address field to index based on the `sizeof()` the inputted data type. 

I did this so that the indexing behaves just like indexing into an array of any type. This means that you will not have to keep track of the `sizeof()` your datatypes when you want to iterate through the data as if it where an array. 

However, just like an array, you will need to keep the data type consistant throughout to keep the index aligned.   

You can expect these two examples to index the exact same way. 
```c++
uint16_t array[5] = {1000, 2000, 3000, 4000}; 

uint16_t number = 1000; 

for (int i = 0; i < 5; i++) {
  array[i] = number * i; 
}
```
```c++

uint16_t number = 1000; 

for (int i = 0; i < 5; i++) {
  PROM.put(0x00 + i, number * i)
}
```

In this example: 
- The index of 0 points to the address `0x00` in the EEPROM. 
- The index of 1 points to the address `0x06` in the EEPROM. 

This is because each write will write three times for redundancy, and the data type being written is 2 bytes wide. 

`2 * 3 = 6.`

#### **Subscript operator: `PROM[address]`** 

This operator allows using the identifier `PROM` like an array.  
EEPROM cells can be read _and_ **_written_** directly using this method.

This operator returns a reference to the EEPROM cell.

```c++
unsigned char val;

//Read first EEPROM cell.
val = PROM[ 0 ];

//Write first EEPROM cell.
PROM[ 0 ] = val;

//Compare contents
if( val == PROM[ 0 ] ){
  //Do something...
}
```

#### **`PROM.length()`**

This function returns an `unsigned int` containing the number of cells in the EEPROM. The value returned is divided by the number of duplicate writes that the library utilizes. 

If the total number of byte sizes cells in the EEPROM is 1024, then the PROM.length() call will yield the result 341, since the default number of redundant writes in the library is 3. Note that the value returned is the result after integer math, and any float remainders are truncated. 

---

### **Advanced features**

This library uses a component based approach to provide its functionality. This means you can also use these components to design a customized approach. Two background classes are available for use: `EERef` & `EEPtr`.

#### **`PRef` class**

This object references an EEPROM cell.
Its purpose is to mimic a typical byte of RAM, however its storage is the EEPROM.
This class has an overhead of two bytes, similar to storing a pointer to an EEPROM cell.

```C++
PRef ref = PROM[ 10 ]; //Create a reference to 11th cell.

ref = 4; //write to EEPROM cell.

unsigned char val = ref; //Read referenced cell.
```

#### **`PPtr` class**

This object is a bidirectional pointer to EEPROM cells represented by `EERef` objects.
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

If for whatever reason you would like to increase the number of redundant writes, then you can change the member variable `PROM.duplicates`. Note that you need at least three for the read function to properly be able to compare each stored value and determine which value is valid. 

```
PROM.duplicates = 4; 
```