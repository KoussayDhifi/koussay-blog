---
title: SHA-1 Implementation in C programming language
description: This post delves into the low level aspect of SHA-1 hash function, so instead of dealing with it as a black box function, this time we're going to know what does this function exactly do.
author: Koussay Dhifi
categories: [Cryptography, Hash Functions, C Programming]
tags: [Cryptography]
pin: true
math: true
mermaid: true
---

## Notes Before Starting

- ***SHA-1 Algorithm is considered unsafe and vulenrable due to known vulnerabilities. This article is for educational purposes only.***
- ***This implementation may contain memory leaks or imperfections. It's built for educational purposes, not production use. Contributions or improvements are welcome!***
- The C code is available in this [github repo](https://github.com/KoussayDhifi/SHA-1).

## What is a hash function

A hash function takes in input a specific string and returns a fixed-length-hash that varies depending on the inserted string

## Attributes of a hash function

- Pseudo-random : The output seems to have a random nature
- Collision-resistant : It is computationally and practically impossible to find two different inputs that produce the same hash output. In other words an ideal hash function can't have two inputs pointing to the same hash.
- Fixed length : For SHA-1 it is 160-bits
- Not reversible : From the output of the hash function you can't determine its original input.

## SHA-1 function

SHA-1 (Secure Hash Algorithm), developed by the **NSA** in **1993**.
It takes in input a string and returns a 160-bit length pseudo-random string, typically displayed as a 40-character hexadecimal string.

### Steps of a hash function

Each hashing function is composed of  four main stages:
```
Message Encoding → Preprocessing → Hash Computations → Message Digest Decoding
```

### Message encoding

Message encoding consists of converting the input string from its format to its binary representation. In other words it is like that
```
Input : "String"
S in ascii is 83 so we convert it to 01010011
S = 01010011  
t = 01110100  
r = 01110010  
i = 01101001  
n = 01101110  
g = 01100111  
Message encoding : 01010011 01110100 01110010 01101001 01101110 01100111
```
This is indeed crucial for the hash function since it relies on low-level **bitwise operators**.

### Preprocessing Phase

This phase consists of padding the message to be suitable for hashing, parsing the padded message into m-bit blocks and setting initialization values to be used in the hash computation.

The following diagram explains this phase

```
Padding → Parsing
```

#### Padding in the preprocessing phase

Padding is basically adding additional bits to the original input to ensure that the message length **in bits** is a multiple of 512.
The padding can be inserted before or during the hash computation.


1- First step in padding

Append 1 bit equals to `1` to the left of the coded message followed by **k** zeros, **k** should verify the following condition:
```
l+1+k = 448 mod (512)
```
Where **l** is the length of the coded message in bits.

2- Second step

Append 64 bits that are equal to **l** converted to binary to the right of the coded message after of course appending those previous **k+1** bits.
In our case the length of string in bits is `l = 8*6 = 48` so we convert 48 to binary
```
(48)10 = (0000...110000)2
```

So the result of the padded message will be the following
```
[Binary message] 1 [k zeros] [64-bit binary of l]
```

#### Parsing the padded message

After padding the coded message -from now on we will refer to it as **M**-, it will be parsed into **N** 512-bit blocks
```
M(1), M(2), M(3), ... M(N)
```

Each block is divided into 16 32-bit words
```
M(i)(0), M(i)(1), ... M(i)(15)
```
So the whole message M is composed of N blocks and each block is composed of 16 words.

### Constants and Initial Hash Values

This section is not a phase of the hash function but to introduce some variables and constants that will be used in the last step of the algorithm which is the hash compuation.

#### Initial Hash Values

The sha-1 function in the end is composed of **five** concatenated **32-bit** variables like this
```
SHA-1 =  (H₀, H₁, H₂, H₃, H₄)
```
Those variables are the final result of the message digest. But they have an initial value at first at the starting of the algorithm the following are the initial values in **hexadecimal** representation:

```
H₀ = 0x67452301  
H₁ = 0xefcdab89  
H₂ = 0x98badcfe  
H₃ = 0x10325476  
H₄ = 0xc3d2e1f0 
```

#### Constants Values

During the hash computations we're going to use **four** constants K(t). While t is the number of iteration in the hashing computation loop. The following are the constants in **hexadecimal** representation

```s
k(0 ≤ t ≤ 19) = 0x5a827999 
k(20 ≤ t ≤ 39 ) = 0x6ed9eba1 
k(40 ≤ t ≤ 59) = 0x8f1bbcdc  
k(60 ≤ t ≤ 79) = 0xca62c1d6   
```

**Note**: In the algorithm we need to convert the initial hash values and the constants from this hexadecimal format to their binary format.

### Hash computations

This phase consists of generating a message scheduler (a function that helps us in extracting a word of the padded message) from the padded message and uses that scheduler with functions, constants and bitwise operations to iteratevly generate a series of hash values. Then the final hash will be the result of the hashing function which is basically called **Message Digest**.


#### Bitwise Operations

Before we get into message scheduling and the algorithm we need to know the bitwise operations we're going to use in this algorithm.

The bitwise operators we're going to use are the known ones `AND`, `NOT`, `OR` and `XOR`.
Adding to those there are other bitwise operators that we're going to use and they will be explained in this article since they are not very known amongst beginners.


**Bitwise right shift operator**

This bitwise operator has the notation of `x >> n` where **x** is the binary we're applying the operation on and **n** is the number of right shifts. Basically this operator shifts bits of **x** to the right by **n** positions like this:

```s
x = 1101
x >> 1 = 0110
x >> 2 = 0011
```

**Bitwise left shift operator**

This bitwise operator is the reverse of the right shift operator it has the notation of `x << n` where **x** is the binary we're applying the operation on and **n** is the number of left shifts. Basically this operator shifts bits of **x** to the left by **n** positions like this:

```s
x = 1101
x << 1 = 1010
x << 2 = 0100
```

**Rotate left operator**

This operator is a combination between **left shift** and **right shift**. It has the notation of `ROTL (x,n)` where **x** is the binary we're applying the operation on and **n** is the number of rotations. Basically this operator rotates **x** by **n** times, by left shifting **n** positions and the left shifted bits are appended to the right of **x**. As shown in the following formula

```
ROTL(x,n)=((x ≪ n) OR ( x ≫ (w−n)))
```

Practical example:

```s
x = 11001010
n = 1
ROTL (x , n) = 10010101
n = 2
ROTL (x, n) = 00101011
n = 3
ROTL (x, n) = 01010110
```

**Modulus Addition**


This operator makes an addition between binary numbers following this specific formula.

```math
(A+B) % 2^w
```

Where **A** and **B** are the decimal representation of two binary numbers. Later we encode the result into binary, and `0 < A < 2^w` and `0 < B < 2^w`. In other words **w** is the number of bits of **A** and **B**.

For SHA-1 in our case numbers are composed of **32 bits** so ` w = 32 `

```math
(A+B) % 2^32
```

**Note**: It will have a normal addition notation later `A+B`


**Nonlinear Function**

This function has the notation of `f (x, y, z, t)` it uses different complex bitwise operators on the variables **x**, **y** and **z** depending on the value of **t**.

The follwoing pseudo-code explains how the function works and what it does return.

```
define f (x, y, z, t)
{
    if 0 <= t <= 19:
        return CH (x, y, z) = (x AND y) XOR (NOT x AND z)
    
    elif 20 <= t <= 39:
        return PARITY (x, y, z) = x XOR y XOR z
    
    elif 40 <= t <= 59:
        return MAJ (x, y, z) = (x AND y) XOR (x AND y) XOR (y AND z)
    
    elif 60 <= t <= 79:
        return PARITY (x, y, z)
}


```

#### Message Scheduler
The message scheduler is a function like the following:

```
w(i)(t) = ( 0<= t <= 15)? M(i)(t) : ROTL( w(i)(t-3) XOR w(i)(t-8) XOR w(i)(t-14) XOR w(i)(t-16), 1 )
```

This function is used to extract a specific **word** from the padded message.

#### Algorithm Of Hash Computations

The following pseudo-code shows exactly the steps of how the hashing computation will work.

```
For i=1 to N
{
    // Intizialise working variables

    a = H₀
    b = H₁ 
    c = H₂ 
    d = H₃ 
    e = H₄

    //Operations on working variables

    For t=0 to 79 
    {
        T = ROTL(a, 5) + f(b, c, d, t) + e + K(t) + w(i)(t)

        e = d
        d = c
        c = ROTL(b, 30)
        d = a
        a = T
    }

    // Initializing new values

    H₀ = a + H₀
    H₁ = b + H₁ 
    H₂ = c + H₂ 
    H₃ = d + H₃ 
    H₄ = e + H₄

}

//Result of SHA-1 in binary

messageDigestInBinary = (H₀, H₁, H₂, H₃, H₄)

```

### Message Digest Decoding

This phase is the simplest all it does it converting the messageDigest from its binary format to the hexadecimal format that we know of SHA-1

```
messageDigest = BinaryToHexa (messageDigestInBinary) 
```

And with that the algorithm of SHA-1 is done




## Building the function in C

This section will be dedicated for building the whole function in C. So if you didn't understand some previous theoratical aspects like **Message Encoding/**, **Preprocessing**, **Hash computations**, **Message Scheduler** and **Message Digest Decoding** . 

This will be a practical example of everything mentionned before.

### C project structure

The structure of the C project will be the following:
```
SHA-1/
├── .gitignore              # Ignores object files, executable, and text files
├── Makefile                # Build configuration
├── README.md               # Project description
├── docs/                   # Documentation directory
│   ├── architecture.md     # Architecture documentation
│   └── design.md           # Design documentation
├── include/                # Header files
│   ├── calcPaddedSize.h    # Calculate padding size function declaration
│   ├── decode.h            # Decoding function declarations
│   ├── encode.h            # Encoding function declarations
│   ├── functions.h         # SHA-1 helper function declarations
│   ├── hashComputation.h   # Core hash computation declarations
│   ├── logicalOperators.h  # Logical operation declarations
│   ├── padding.h           # Message padding function declarations
│   └── readingInput.h      # Input handling function declarations
├── src/                    # Source files
│   ├── calcPaddedSize.c    # Padding size calculation implementation
│   ├── decode.c            # Binary to hex/char decoding implementation
│   ├── encode.c            # Text to binary encoding implementation
│   ├── functions.c         # SHA-1 helper functions implementation
│   ├── hashComputation.c   # Core hash computation implementation
│   ├── logicalOperators.c  # Logical operations implementation
│   ├── main.c              # Main program entry point
│   ├── padding.c           # Message padding implementation
│   └── readingInput.c      # Input handling implementation
└── build/                  # Build output directory (implied from Makefile)
    └── *.o                 # Object files (generated during build)
```

This tree contains everything we will create throughout this journey.

This is a good structure for any C project, so if you never saw such structure it is an opportunity to learn from it.

### Main C file

This section is about the main file. This file basically sums up all the steps that a hash function does.
So to have a broader image on what next functions we're going to implement next in C.

The following is the implementation of `main.c` file.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Including all necessary headers for different SHA-1 stages
#include "../include/readingInput.h"
#include "../include/encode.h"
#include "../include/decode.h"
#include "../include/calcPaddedSize.h"
#include "../include/padding.h"
#include "../include/functions.h"
#include "../include/hashComputation.h"

// Initializing SHA-1 hash values (in hexadecimal)
#define H0 "67452301"
#define H1 "efcdab89"
#define H2 "98badcfe"
#define H3 "10325476"
#define H4 "c3d2e1f0"

// Constant lengths used in the algorithm
#define LENGTHHEXADEC 40
#define LENGOFHASH 160
#define LENGTHWORKINGVARIABLES 8
#define LENGTHWORKINGVARIABLESCONVERSION 32


// Function to print a char array (final hex result)
void showArrayChar(char* T, int n) {
  for (int i = 0; i < n; i++) {
    printf("%c", T[i]);
  }
}

// Entry point of the program
void main(int argc, char** argv) {
  char* msg;

  // Step 1: Read user input (either from a string or a file)
  readingInput(argc, argv, &msg);

  // Step 2: Get message length in bytes and bits
  // Why? : To determine the size of the array
  const size_t msgLength = strlen(msg);
  const size_t msgLengthInBits = 8 * msgLength;

  // Step 3: Allocate and perform binary encoding of the message
  int* codedMsg = (int*) malloc((msgLengthInBits + 1) * sizeof(int));
  encode(msg, codedMsg);

  // Step 4: Determinating padding size and allocate memory for the padded message
  const size_t paddedMsgSize = calcPaddedSize(msgLengthInBits);
  int* paddedMsg = (int*) malloc((paddedMsgSize + 1) * sizeof(int));
  
  // Step 5: Padding the encoded message
  padding(codedMsg, msgLengthInBits, paddedMsg, paddedMsgSize);

  // Step 6: Count the number of 512-bit blocks
  int numberOfBlocks = paddedMsgSize / 512;

  // Step 7: Prepare result array and initial working variables
  int resultOfHash[LENGOFHASH];
  int workingVariables[5][32]; // Holds A, B, C, D, E

  // Temporary variables to hold binary versions of initial hash values (H₀, H₁, H₂, H₃, H₄)
  int codedMsgOne[32], codedMsgTwo[32], codedMsgThree[32], codedMsgFour[32], codedMsgFive[32];

  // Step 8: Convert initial H₀-H₄ from hex to 32-bit binary
  hex2Binary(H0, LENGTHWORKINGVARIABLES, codedMsgOne, LENGTHWORKINGVARIABLESCONVERSION);
  hex2Binary(H1, LENGTHWORKINGVARIABLES, codedMsgTwo, LENGTHWORKINGVARIABLESCONVERSION);
  hex2Binary(H2, LENGTHWORKINGVARIABLES, codedMsgThree, LENGTHWORKINGVARIABLESCONVERSION);
  hex2Binary(H3, LENGTHWORKINGVARIABLES, codedMsgFour, LENGTHWORKINGVARIABLESCONVERSION);
  hex2Binary(H4, LENGTHWORKINGVARIABLES, codedMsgFive, LENGTHWORKINGVARIABLESCONVERSION);

  // Step 9: Copy initial hash values into working variables
  copyArray(codedMsgOne, workingVariables[0], 32);
  copyArray(codedMsgTwo, workingVariables[1], 32);
  copyArray(codedMsgThree, workingVariables[2], 32);
  copyArray(codedMsgFour, workingVariables[3], 32);
  copyArray(codedMsgFive, workingVariables[4], 32);

  // Step 10: Hashing Computation
  hashComputation(numberOfBlocks, paddedMsg, paddedMsgSize, resultOfHash, LENGOFHASH, workingVariables);

  // Step 11: Message Digest Decoding
  char hexaDecimalResult[LENGTHHEXADEC];
  binary2Hex(resultOfHash, LENGOFHASH, hexaDecimalResult, LENGTHHEXADEC);

  // Step 12: Print the result
  printf("SHA1 = ");
  showArrayChar(hexaDecimalResult, LENGTHHEXADEC);
  printf("\n");


}

```


### Reading input

In this section we will focus on how the hash function in C will take input by discussing **the input format**, **message encoding** and **message decoding**.

#### Input format

The input is going to be a **command line argument** inserted by the user, in this format
```
$ ./sha-1 -[sf] <input>
```

There will be two command line arguments.
- The flag that indicates the input type 
- The input

The input can be either a string like this:
```$ ./sha-1 -s "This is a string"```
or a file if we chose the f flag:
```$ ./sha-1 -f file_path```

#### If the input is a string

The string will be parsed and stored in a **string** data structure in C or to be more specific in an **array of characters** in C, each character will have its own array index like this:

```s
$ ./sha-1 -s "This"
The C program will parse it into this : T = {'T', 'h', 'i', 's'}

```

#### If the input is a file (path)

The program reads the file, stores it into a variable and then parses it the same way as it parses a string.


##### Implementation in C

The following is the full implementation of reading the input in C.

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define FILE_FLAG "-f"
#define STRING_FLAG "-s"

// This function reads the input from either a string or a file
// based on command-line arguments, and stores it in 'msg'.
void readingInput(int argc, char** argv, char** msg) {
  
    // Ensure the correct number of arguments is provided
    if (argc != 3) {
        fprintf(stderr, 
            "Uncompatible number of positional arguments. Expected 2, got %d.\n"
            "Hint: If your input has spaces, wrap it in quotes (\"\").\n", 
            argc - 1);
        exit(EXIT_FAILURE);
    }

    // Handle string input (e.g., ./sha-1 -s "Hello")
    if (strcmp(argv[1], STRING_FLAG) == 0) {
        *msg = argv[2];  // Point to the user-provided string directly

    // Handle file input (e.g., ./sha-1 -f file.txt)
    } else if (strcmp(argv[1], FILE_FLAG) == 0) {

        // Open file in read mode
        FILE* filePtr = fopen(argv[2], "r");
        if (filePtr == NULL) {
            fprintf(stderr, "Failed to open file: %s\n", argv[2]);
            exit(EXIT_FAILURE);
        }

        // Move file pointer to the end to determine file size
        if (fseek(filePtr, 0, SEEK_END) != 0) {
            fprintf(stderr, "Could not determine the size of file. Exiting...\n");
            fclose(filePtr);
            exit(EXIT_FAILURE);
        }

        long fileSize = ftell(filePtr);  // Get current file pointer position (i.e., size)
        rewind(filePtr);                 // Move pointer back to the beginning

        // Allocate memory to hold the entire file content
        *msg = (char*) malloc(fileSize + 1);  // +1 for null terminator
        if (*msg == NULL) {
            fprintf(stderr, "Memory allocation failed.\n");
            fclose(filePtr);
            exit(EXIT_FAILURE);
        }

        // Read file content into the buffer
        size_t bytesRead = fread(*msg, 1, fileSize, filePtr);
        if (bytesRead != fileSize) {
            fprintf(stderr, "Error reading file. Exiting...\n");
            free(*msg);
            fclose(filePtr);
            exit(EXIT_FAILURE);
        }

        (*msg)[fileSize] = '\0';  // Null-terminate the buffer
        fclose(filePtr);          // Close the file

    } else {
        // If an unknown flag is provided
        fprintf(stderr, "Unrecognized flag: %s\n", argv[1]);
        exit(EXIT_FAILURE);
    }
}

```

### Message Encoding/Decoding

This section takes care of a frequently used functions during this C project which are the `encode`, `decode`, `hex2Binary` and `binary2Hex` functions
It takes in parameter an array of characters and converts the string to its binary representation as we discussed in the beginning of the article


#### Encode function

The following is the implementation of the encode function

```c


// Converts a character 'c' to its binary representation.
// Stores the result as an array of 0s and 1s in 'res', most significant bit first.
void char2Binary (char c, int* res) {
  int charValue = c;  // Get ASCII value of character

  // Convert to binary by dividing by 2 and storing the remainder
  for (int i = 0; i < NUMBEROFBITS; i++) {
    *(res + NUMBEROFBITS - 1 - i) = (charValue % 2 == 1);
    charValue /= 2;
  } 
}

// Encodes a string message into its binary representation.
// Each character is converted to 8 bits (ASCII binary), and stored in codedMsg.
void encode (char* msg, int* codedMsg) {
  
  size_t sizeMsg = strlen(msg);  // Get the length of the input string
  int j = 0;                     // Index for the codedMsg array
  int res[NUMBEROFBITS];        // Temporary array to store binary of a single character

  for (int i = 0; i < sizeMsg; i++) {
    char2Binary(*(msg + i), res);  // Convert character to 8-bit binary
    
    // Copy the 8-bit result into the codedMsg array
    for (int k = 0; k < NUMBEROFBITS; k++) {
      *(codedMsg + j) = *(res + k);
      j++;
    }
  }
}




```

#### Decode function

The following code is the implementation of the decode function and its dependancies

```c
// Converts an 8-bit binary array 'binary' to a corresponding character 'c'.
// The binary array is interpreted from most significant bit (left) to least significant (right).
void binary2Char(int* binary, char* c) {
  int power = 7;  // Start with the highest bit (2^7 = 128)
  *c = 0;  // Initialize character to 0

  for (int i = 0; i < 8; i++) {
    // Set the appropriate bit in the character based on the binary array
    *c += (*(binary + i)) ? (1 << power) : 0;
    power--;  // Move to the next lower bit
  }
}

// Decodes a binary message 'codedMsg' of size 'codedMsgSize' into the decoded message 'decodedMsg'.
// The binary data is decoded into characters 8 bits at a time.
void decode(int* codedMsg, size_t codedMsgSize, char* decodedMsg) {
  int aux[8];  // Temporary array to hold 8 bits for a character
  int j = 0;  // Index for storing the decoded characters
  int auxCounter = 0;  // Index for the 'aux' array

  for (int i = 0; i < codedMsgSize; i++) {
    *(aux + auxCounter) = *(codedMsg + i);  // Store the current bit in 'aux'
    auxCounter++;

    if ((i + 1) % 8 == 0) {  // After 8 bits (one character)
      binary2Char(aux, decodedMsg + j);  // Convert 8 bits to a character
      j++;  // Move to the next position in decodedMsg
      auxCounter = 0;  // Reset the auxiliary counter
    }
  }

}


```

#### Hex2Binary function

The following code is the implementation of hex2Binary function which converts binary array to its hexadecimal representation

```c
// Converts a decimal number to its binary representation of length 'n'.
// The result is stored in 'binary' array with the most significant bit first.
void number2Binary(uintmax_t number, int* binary, int n) {
  for (int i = 0; i < n; i++) {
    *(binary + n - 1 - i) = (number % 2 == 1);
    number /= 2;
  }
}

// Returns the integer value of a hexadecimal character.
// Supports characters '0'-'9' and 'a'-'f'.
int correspondingNumberHex (char c) {
  if (c >= '0' && c <= '9') {
    return c - '0';
  } else {
    return c - 'a' + 10;
  }
}

// Converts a hexadecimal string 'hexArray' of length 'lengthOfHex'
// into a binary array of length 'lengthOfBinaryArray'.
// Each hex digit becomes 4 binary digits.
void hex2Binary (char* hexArray, int lengthOfHex, int* binaryArray, int lengthOfBinaryArray) {
  int binaryFormat[LENGTHOFCONVERSION] = {0};  // Temporary array for 4-bit binary
  int binaryArrayIndex = 0;

  for (int i = 0; i < lengthOfHex; i++) {
    int decimalNumber = correspondingNumberHex(*(hexArray + i));
    number2Binary(decimalNumber, binaryFormat, LENGTHOFCONVERSION);

    for (int j = 0; j < LENGTHOFCONVERSION; j++) {
      *(binaryArray + binaryArrayIndex++) = *(binaryFormat + j);
    }
  }
}
```

#### Binary2hex function

The following code is the implementation of the binary2Hex function that converts binary to hexadecimal and its dependancies.

```c
#define LENGTHOFCONVERSION 4  // Defines the length of each binary group for hexadecimal conversion (4 bits)


// Converts a binary array 'x' of size 'n' into a decimal number (uintmax_t).
uintmax_t binary2Number(int* x, int n) {
  int power = n - 1;  // Start from the highest power (2^(n-1))
  uintmax_t res = 0;  // Initialize the result

  // Iterate through the binary array and convert each bit into its corresponding decimal value
  for (int i = 0; i < n; i++) {
    res += (*(x + i)) ? (1 << power) : 0;  // Add the bit value to the result
    power--;  // Decrease the power for the next bit
  }

  return res;  // Return the final decimal result
}


// Converts a decimal number (0-15) to its corresponding hexadecimal character.
char decimal2Hex(int decimal) {
  if (decimal < 10) {
    return '0' + decimal;  // For decimal numbers 0-9, return the character '0'-'9'
  } else {
    return 'a' + (decimal - 10);  // For decimal numbers 10-15, return 'a'-'f'
  }
}


// Converts a binary array 'binaryArray' of length 'lengthOfBinaryArray' into a hexadecimal string 'hexConv'.
// The length of the hexadecimal string is specified by 'lengthOfHex'.
void binary2Hex(int* binaryArray, int lengthOfBinaryArray, char* hexConv, int lengthOfHex) {
  int aux[LENGTHOFCONVERSION];  // Temporary array to hold a group of 4 binary bits
  int j = 0;  // Index for storing bits in 'aux'
  int hexCounter = lengthOfHex - 1;  // Start filling the hexadecimal string from the end
  
  // Iterate over the binary array from the last bit to the first
  for (int i = lengthOfBinaryArray - 1; i >= 0; i--) {
    aux[LENGTHOFCONVERSION - 1 - j++] = *(binaryArray + i);  // Store the current bit in 'aux'
    
    // After every 4 bits, convert the binary group to hexadecimal
    if (i % 4 == 0) {
      int decimalValue = binary2Number(aux, LENGTHOFCONVERSION);  // Convert 4 binary bits to a decimal number
      *(hexConv + hexCounter--) = decimal2Hex(decimalValue);  // Convert the decimal value to hex and store in 'hexConv'
      j = 0;  // Reset 'aux' index for the next group
    }
  }
}
```

### Calculating the padding size

This function will help us in solving the equation
```
l+1+k = 448 mod (512)
```
To know the total length of the padded msg.

The following is the full implementation in C.

```c
#include <stdio.h>

// Calculate the padded size of a binary message for SHA-like padding.
int calcPaddedSize(int codedMsgLength) {
  // k is the number of 0s needed to make the final size ≡ 448 mod 512 (i.e., 64 bits remaining for the length)
  int k = (512 - ((codedMsgLength + 65) % 512)) % 512;

  // Final size is original length + 1 ('1' bit) + k ('0' bits) + 64 (length field)
  return codedMsgLength + 65 + k;
}


```


### Message Padding

This section will take care of padding the message function is C.
The following is the full function responsible for padding the message implemented in C.

```c
#include <stdio.h>
#include <stdlib.h>
#include "../include/encode.h"

#define FIRST_PADDING 1       // First bit after the message is set to 1
#define LATER_PADDING 0       // Remaining padding bits (after FIRST_PADDING) are 0
#define LENGTHMSG 64          // Message length is encoded using 64 bits

// This function applies SHA-like padding to a binary message.
void padding(int* codedMsg, size_t codedMsgSize, int* paddedMsg, size_t paddedMsgSize) {
  
  int binary[LENGTHMSG] = {0};  // Array to store the 64-bit binary representation of the original length

  // Convert the message length into a 64-bit binary and store it in `binary`
  number2Binary(codedMsgSize, binary, LENGTHMSG);

  int j = 0;  // Index for copying bits from `binary` into `paddedMsg`

  for (int i = 0; i < paddedMsgSize; i++) {

    if (i < codedMsgSize) {
      // Copy the original message bits
      *(paddedMsg + i) = *(codedMsg + i);

    } else if (i == codedMsgSize) {
      // Add the first padding bit (a single '1')
      *(paddedMsg + i) = FIRST_PADDING;

    } else if (i <= paddedMsgSize - LENGTHMSG - 1) {
      // Fill with zeros until the last 64 bits
      *(paddedMsg + i) = LATER_PADDING;

    } else {
      // Append the 64-bit length of the original message at the end
      *(paddedMsg + i) = *(binary + j);
      j++;
    }
  }
}

```

### Bitwise Operators Implementation

This section will delve into the implementation of bitwise operators used in SHA-1.

#### AND, OR, XOR, NOT

The following is the implementation of the 3 bitwise operators `AND`, `OR`, `NOT` and `XOR` in C

```c
#include <stdio.h>
#include <stdlib.h>
#include "../include/logicalOperators.h"

#define BLOCKSIZE 32  // Each binary block is 32 bits

// Macros for declaring and allocating memory for results
#define DECLARE(result) int* result = NULL;
#define ALLOCATE(result)  \
  result = (int*) malloc(BLOCKSIZE * sizeof(int)); \
  if (result == NULL) { \
    fprintf(stderr, "Failed to allocate memory, aborting ... \n"); \
    exit(EXIT_FAILURE); \
  }

// Logical NOT: result[i] = !binary[i]
int* logicalNOT(int* binary) {
  DECLARE(result);
  ALLOCATE(result);

  for (int i = 0; i < BLOCKSIZE; i++) {
    *(result + i) = !*(binary + i);
  }
  return result;
}

// Logical AND: result[i] = x[i] & y[i]
int* logicalAND(int* x, int* y) {
  DECLARE(result);
  ALLOCATE(result);

  for (int i = 0; i < BLOCKSIZE; i++) {
    *(result + i) = *(x + i) & *(y + i);
  }

  return result;
}

// Logical XOR: result[i] = x[i] ^ y[i]
int* logicalXOR(int* x, int* y) {
  DECLARE(result);
  ALLOCATE(result);

  for (int i = 0; i < BLOCKSIZE; i++) {
    *(result + i) = *(x + i) ^ *(y + i);
  }

  return result;
}

// Logical OR: result[i] = x[i] | y[i]
int* logicalOR(int* x, int* y) {
  DECLARE(result);
  ALLOCATE(result);

  for (int i = 0; i < BLOCKSIZE; i++) {
    *(result + i) = *(x + i) | *(y + i);
  }

  return result;
}
```

#### Right shift operator
The following is the implementation of the right shift operator in C


```c

#define WORDSIZE 32
#define ZERO 0

// Performs a logical right shift of 'x' by 'n' bits and stores the result in 'res'
void rightShift(int* x, int n, int* res) {
  for (int i = 0; i < WORDSIZE; i++) {
    if (i < n)
      *(res + i) = ZERO;            // Fill leftmost positions with 0
    else
      *(res + i) = *(x + i - n);    // Shift bits to the right
  }
}


```

#### Left shift operator

The following is the implementation of the left shift operator in C

```c
#define WORDSIZE 32
#define ZERO 0

// Performs a logical left shift of 'x' by 'n' bits and stores the result in 'res'
void leftShift(int* x, int n, int* res) {
  for (int i = 0; i < WORDSIZE; i++) {
    if (i + n < WORDSIZE)
      *(res + i) = *(x + i + n);  // Shift bits to the left
    else
      res[i] = ZERO;  // Fill shifted-out positions with 0
  }
}

```

#### Rotate left operator

The following is the implementation of the rotate left operator

```c

#define WORDSIZE 32
#define ZERO 0

void ROTL (int* x, int n, int* res) {
  
  int* xLeftShifted = (int*) malloc(WORDSIZE * sizeof(int));
  leftShift(x, n, xLeftShifted);
  
  

  int* xRightShifted = (int*) malloc(WORDSIZE * sizeof(int));
  rightShift(x, WORDSIZE - n, xRightShifted);
  
  int* leftOrRight = logicalOR(xLeftShifted, xRightShifted);

  for (int i = 0; i<WORDSIZE; i++) {
    *(res + i) = *(leftOrRight + i);
  }
  

  free (leftOrRight);
  free (xRightShifted);
  free (xLeftShifted);

} 
```

#### Nonlinear Function implementation

The following code represents the implementation of the non linear function containing different bitwise operators like `CH`, `PARITY` and `MAJ`

```c

#define WORDSIZE 32

// Implements the SHA-1 CHOICE function: CH(x, y, z) = (x AND y) XOR (NOT x AND z)
void CH(int* x, int* y, int* z, int* res) {
  int* logicalAnd = logicalAND(x, y);         // x AND y
  int* not = logicalNOT(x);                   // NOT x
  int* logicalAndTwo = logicalAND(not, z);    // NOT x AND z
  int* logicalxor = logicalXOR(logicalAnd, logicalAndTwo); // (x AND y) XOR (NOT x AND z)

  // Copy result to output array
  for (int i = 0; i < WORDSIZE; i++) {
    *(res + i) = *(logicalxor + i); 
  }

  // Free allocated memory
  free(logicalxor);
  free(logicalAndTwo);
  free(logicalAnd);
  free(not);
}

// Implements the SHA-1 PARITY function: PARITY(x, y, z) = x XOR y XOR z
void PARITY(int* x, int* y, int* z, int* res) {
  int* logicalxor = logicalXOR(x, y);           // x XOR y
  int* logicalxorTwo = logicalXOR(logicalxor, z); // (x XOR y) XOR z

  // Copy result to output array
  for (int i = 0; i < WORDSIZE; i++) {
    *(res + i) = *(logicalxorTwo + i);
  }

  // Free allocated memory
  free(logicalxorTwo);
  free(logicalxor);
}

// Implements the SHA-1 MAJORITY function: MAJ(x, y, z) = (x AND y) XOR (x AND z) XOR (y AND z)
void MAJ(int* x, int* y, int* z, int* res) {
  int* logicalAndOne = logicalAND(x, y);     // x AND y
  int* logicalAndTwo = logicalAND(x, z);     // x AND z
  int* logicalAndThree = logicalAND(y, z);   // y AND z
  int* logicalxor = logicalXOR(logicalAndOne, logicalAndTwo); // (x AND y) XOR (x AND z)
  int* logicalxorTwo = logicalXOR(logicalxor, logicalAndThree); // ((x AND y) XOR (x AND z)) XOR (y AND z)

  // Copy result to output array
  for (int i = 0; i < WORDSIZE; i++) {
    *(res + i) = *(logicalxorTwo + i);
  }

  // Free allocated memory
  free(logicalxorTwo);
  free(logicalxor);
  free(logicalAndOne);
  free(logicalAndTwo);
  free(logicalAndThree);
}

// Dispatches the appropriate SHA-1 function based on round index t
// SHA-1 uses different functions depending on the round:
// t in [0,19]   → CH
// t in [20,39] or [60,79] → PARITY
// t in [40,59]  → MAJ
void functions(int* x, int* y, int* z, int t, int* res) {
  if (t >= 0 && t <= 19) {
    CH(x, y, z, res);
  } else if ((t >= 20 && t <= 39) || (t >= 60 && t <= 79)) {
    PARITY(x, y, z, res);
  } else if (t >= 40 && t <= 59) {
    MAJ(x, y, z, res);
  } else {
    // Invalid round index
    fprintf(stderr, "Error: t must be in [0 .. 80), got %d\n", t);
    exit(EXIT_FAILURE);
  }
}
```

#### Modulus Addition Implementation

The following is the C code of the modulus addition function.

```c

#define WORDSIZE 32

void modulusAddition(int* x, int* y, int* res, int w) {
  
  // Convert the binary arrays x and y to their uintmax_t numeric representations
  uintmax_t X = binary2Number(x, WORDSIZE);
  uintmax_t Y = binary2Number(y, WORDSIZE);
  
  // Perform the modular addition (X + Y) mod 2^w
  uintmax_t Z = (X + Y) % (1ULL << w);
  
  // Convert the result Z back to binary and store it in res
  number2Binary(Z, res, WORDSIZE); // Already Explained in the encoding section
}

```

### Message Scheduler Implementaion

This section takes care of the message scheduler and the following is the implementation of the message scheduler code with its dependancies.

```c
#define WORDSIZE 32
#define BLOCKSIZE 512
#define ZERO 0
#define ITERATIONS 80
#define NUMBEROFSHIFTS 1

// Function to divide the padded message into blocks for processing.
void blockDivider (int* paddedMsg, int res[32], int t, int numberOfBlock, int paddedMsgSize, size_t resSize) {
  
  // Calculate the starting position of the current block
  int startingOfBlock = (numberOfBlock-1)*BLOCKSIZE;
  int block[BLOCKSIZE] = {0};
  
  // Check for invalid block number or block size exceeding the padded message size
  if (numberOfBlock <= 0 || (numberOfBlock - 1)* BLOCKSIZE >= paddedMsgSize) {
    printf("Invalid Block number \n");
    printf("%d\n",numberOfBlock);
    exit(1);
  }

  // Check if the value of 't' is valid for the block size
  if (t * WORDSIZE >= BLOCKSIZE) {
    printf("Invalid T \n");
    exit(1);
  }
  
  // Ensure we don't exceed the padded message size
  if (startingOfBlock + BLOCKSIZE > paddedMsgSize) {
      printf("Buffer overflow");
      exit(1);
  }

  // Copy the appropriate block from the padded message to the 'block' array
  for (int i = startingOfBlock; i<startingOfBlock+BLOCKSIZE; i++) {
    if (i >= paddedMsgSize) {
      printf("Can't access this memory");
      exit(1);
    } else if ((i-startingOfBlock) >= BLOCKSIZE) {
      printf("Problem in block,");
      exit(1);
    } else {
      *(block + (i-startingOfBlock)) = *(paddedMsg + i);
    }
  }
  
  // Calculate the starting position of the word within the block
  int startingOfWord = t * WORDSIZE;

  // Ensure the word does not exceed the block size
  if (startingOfWord + WORDSIZE > BLOCKSIZE) {
    printf("Exceeds block size");
    exit(1);
  }

  // Copy the appropriate word from the block to the result array 'res'
  for (int i = startingOfWord; i<startingOfWord+WORDSIZE; i++) {
    if (i >= BLOCKSIZE) {
      printf("Exceeds block");
      exit(1);
    } else if ((i-startingOfWord) >= 32){
      printf("Exceeds result");
      exit(1);
    } else {
      *(res + (i-startingOfWord)) = *(block + i);
    }
  }
 
  // Ensure the result array size is valid
  if (resSize < WORDSIZE * sizeof(int)) {
    printf("Buffer size mismatch \n");
    exit(1);
  }
}

// Function to display an array of integers
void showArray2 (int* array, int n) {
  for (int i = 0; i<n; i++) 
    printf("%d", array[i]);

  printf("\n");
}

// Message scheduling function for SHA computation
void messageScheduler (int* paddedMsg, int res[32], int t, int n, int paddedMsgSize, size_t resSize) {
  char wordToDebug[8];

  // Check for null input arrays
  if (paddedMsg == NULL || res == NULL) 
    printf("INVALID");

  // Cache for words of the message schedule
  static int wCache [ITERATIONS][WORDSIZE] = {0};
  
  // If 't' is within the first 16 words of the message schedule, get the word from the padded message
  if (t <= 15) {
    // Call the blockDivider function to get the word
    blockDivider(paddedMsg, res, t, n, paddedMsgSize, resSize);
    
    // Cache the word into the wCache array
    for (int i = 0; i<WORDSIZE; i++) {
      wCache[t][i] = res[i];
    }

  } else {
    // For t > 15, calculate the word based on the previous words using logical XOR and rotation
    int* XorOne = logicalXOR(wCache[t-3], wCache[t-8]);
    int* XorTwo = logicalXOR(wCache[t-14], wCache[t-16]);
    int* finalXOR = logicalXOR(XorOne, XorTwo);
    
    // Rotate the final XOR result and store it in the result array
    ROTL(finalXOR, NUMBEROFSHIFTS, res);
    
    // Cache the rotated word
    for (int i = 0; i<WORDSIZE; i++) {
      wCache[t][i] = res[i];
    }
   
    // Free the memory allocated for intermediate XOR results
    free(XorOne);
    free(XorTwo);
    free(finalXOR);
  }
}


```


### Hash Computations Implementation

This function is the implementation of the explained algorithm about hash computations

```c

#include <stdio.h>
#include <stdlib.h>
#include "../include/functions.h"
#include "../include/encode.h"

#define LENGTHWORKINGVAR 32       // Length of working variables in bits (32 bits)
#define LENGTHWORKINGVARHEX 8     // Length of constants in hexadecimal representation
#define NUMBEROFCONSTANTS 4       // Number of SHA constants (K0 to K3)
#define NUMBEROFROTATIONSONE 5    // Number of rotations for one part of the process
#define NUMBEROFROTATIONSTWO 30   // Number of rotations for another part of the process
#define WORKINGVARIABLES 5        // Total number of working variables (a, b, c, d, e)
#define COMPUTATIONS 80           // Number of computations in each round (t from 0 to 79)
#define K0 "5a827999"             // SHA constant K0
#define K1 "6ed9eba1"             // SHA constant K1
#define K2 "8f1bbcdc"             // SHA constant K2
#define K3 "ca62c1d6"             // SHA constant K3


// This function generates an array of constants for SHA hashing
void generateArrayOfSHAConstants (int K[NUMBEROFCONSTANTS][LENGTHWORKINGVAR]) {
  hex2Binary(K0, LENGTHWORKINGVARHEX, K[0], LENGTHWORKINGVAR);  // Convert K0 to binary and store
  hex2Binary(K1, LENGTHWORKINGVARHEX, K[1], LENGTHWORKINGVAR);  // Convert K1 to binary and store
  hex2Binary(K2, LENGTHWORKINGVARHEX, K[2], LENGTHWORKINGVAR);  // Convert K2 to binary and store
  hex2Binary(K3, LENGTHWORKINGVARHEX, K[3], LENGTHWORKINGVAR);  // Convert K3 to binary and store
}

// This function performs the core SHA hash computation over multiple blocks
void hashComputation(int numberOfBlocks, int* paddedMsg, size_t paddedMsgSize, int* resultOfHash, size_t lengthOfHash, int workingVariables[5][32]) {
  
  // Initialize working variables (a, b, c, d, e)
  int a[LENGTHWORKINGVAR] = {0};
  int b[LENGTHWORKINGVAR] = {0};
  int c[LENGTHWORKINGVAR] = {0};
  int d[LENGTHWORKINGVAR] = {0};
  int e[LENGTHWORKINGVAR] = {0};
  
  int K[NUMBEROFCONSTANTS][LENGTHWORKINGVAR];  // Constants for the SHA function
  generateArrayOfSHAConstants(K);  // Generate constants K0, K1, K2, K3
  
  // Temporary variables for the modular sum calculations
  int modularSumOne[LENGTHWORKINGVAR] = {0};
  int modularSumTwo[LENGTHWORKINGVAR] = {0};
  int modularSumThree[LENGTHWORKINGVAR] = {0};
  int modularSumFour[LENGTHWORKINGVAR] = {0};
  int modularSumFive[LENGTHWORKINGVAR] = {0};
  
  int T[LENGTHWORKINGVAR] = {0};          // Temporary result for each round
  int rotationResult[LENGTHWORKINGVAR] = {0};  // Temporary result for rotations
  int functionResult[LENGTHWORKINGVAR] = {0};  // Temporary result for function calculations
  int messageSchedulerResult[LENGTHWORKINGVAR] = {0};  // Temporary result for message scheduling
  int indexOfK;  // Index to select the corresponding SHA constant
  
  // Iterate over the blocks
  for (int i = 0; i < numberOfBlocks; i++) {
    
    // Copy the initial working variables (a, b, c, d, e) from the workingVariables array
    copyArray(workingVariables[0], a, LENGTHWORKINGVAR);
    copyArray(workingVariables[1], b, LENGTHWORKINGVAR);
    copyArray(workingVariables[2], c, LENGTHWORKINGVAR);
    copyArray(workingVariables[3], d, LENGTHWORKINGVAR);
    copyArray(workingVariables[4], e, LENGTHWORKINGVAR);
   
    // Iterate over the 80 rounds (t = 0 to 79)
    for (int t = 0; t < COMPUTATIONS; t++) {

      // Select the SHA constant based on the current round (t)
      if (t <= 19) {
        indexOfK = 0;
      } else if (t >= 20 && t <= 39) {
        indexOfK = 1;
      } else if (t >= 40 && t <= 59) {
        indexOfK = 2;
      } else {
        indexOfK = 3;
      }
      
      // Perform rotation on 'a' with NUMBEROFROTATIONSONE
      ROTL(a, NUMBEROFROTATIONSONE, rotationResult);
      
      // Call the functions to compute the logical operations based on round
      functions(b, c, d, t, functionResult);
      
      // Call the messageScheduler to get the appropriate message block for this round
      size_t size = sizeof(messageSchedulerResult);
      messageScheduler(paddedMsg, messageSchedulerResult, t, i + 1, paddedMsgSize, size);
      
      // Compute the modular addition of various values to produce intermediate results
      modulusAddition(rotationResult, functionResult, modularSumOne, LENGTHWORKINGVAR);
      modulusAddition(e, K[indexOfK], modularSumTwo, LENGTHWORKINGVAR);
      modulusAddition(modularSumOne, modularSumTwo, modularSumThree, LENGTHWORKINGVAR);
      modulusAddition(modularSumThree, messageSchedulerResult, T, LENGTHWORKINGVAR);
      
      // Update the working variables (a, b, c, d, e)
      copyArray(d, e, LENGTHWORKINGVAR);
      copyArray(c, d, LENGTHWORKINGVAR);
      
      // Perform rotation on 'b' with NUMBEROFROTATIONSTWO
      ROTL(b, NUMBEROFROTATIONSTWO, rotationResult);
      copyArray(rotationResult, c, LENGTHWORKINGVAR);
      copyArray(a, b, LENGTHWORKINGVAR);
      copyArray(T, a, LENGTHWORKINGVAR);
    }
    
    // After all rounds, update the working variables with the final modular sums
    modulusAddition(a, workingVariables[0], modularSumOne, LENGTHWORKINGVAR);
    modulusAddition(b, workingVariables[1], modularSumTwo, LENGTHWORKINGVAR);
    modulusAddition(c, workingVariables[2], modularSumThree, LENGTHWORKINGVAR);
    modulusAddition(d, workingVariables[3], modularSumFour, LENGTHWORKINGVAR);
    modulusAddition(e, workingVariables[4], modularSumFive, LENGTHWORKINGVAR);
    
    // Copy the final modular sums back into the working variables array for the next block
    copyArray(modularSumOne, workingVariables[0], LENGTHWORKINGVAR);
    copyArray(modularSumTwo, workingVariables[1], LENGTHWORKINGVAR);
    copyArray(modularSumThree, workingVariables[2], LENGTHWORKINGVAR);
    copyArray(modularSumFour, workingVariables[3], LENGTHWORKINGVAR);
    copyArray(modularSumFive, workingVariables[4], LENGTHWORKINGVAR);
  }
  
  // Combine the final results of the working variables into the result hash
  int k = 0;
  for (int i = 0; i < WORKINGVARIABLES; i++) {
    for (int j = 0; j < LENGTHWORKINGVAR; j++) {
      *(resultOfHash + k) = workingVariables[i][j];
      k++;
    }
  }
}


```

## Testing The function

When we run the function using the simple input "abc" it will give us the follwoing result

```s
$ ./final -s "abc"
SHA1 = a9993e364706816aba3e25717850c26c9cd0d89d

```

## Conclusion And Questions

That was the full implementation of SHA-1 function in C after explaining it and deeping dive into it.
Everything will be found in this [github repo](https://github.com/KoussayDhifi/SHA-1).

But the SHA-1 Algorithm is now regarded as vulenrable and insecure and should not be used. Why? 
In the next article we'll delve into the vulenrabalities of SHA-1 algorithm and why it is considered as dangerous to use.