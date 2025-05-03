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

## What is a hash function

A hash function takes in input a specific string and returns a fixed-length-hash that varies depending on the inserted string

## Attributes of a hash function

- Pseudo-random : The output seems to have a random nature
- Collision-resistant : It is computationally and practically impossible to find two different inputs that produce the same hash output. In other words an ideal hash function can't have two inputs pointing to the same hash.
- Fixed length : For SHA-1 it is 160-bits
- Not reversible : From the output of the hash function you can't determine its original input.

## SHA-1 function

SHA-1 (Secure Hash Algorithm), developed by the **NSA** in **1993**.
It takes in input a string and returns a 160-bit length pseudo-random string.

### Steps of a hash function

Each hashing function is composed of  four main stages:
```
Message Encoding → Preprocessing → Hash Computations → Message Digest Decoding
```

#### Message encoding

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

#### Preprocessing Phase

This phase consists of padding the message to be suitable for hashing, parsing the padded message into m-bit blocks and setting initialization values to be used in the hash computation.

The following diagram explains this phase

```
Padding → Parsing
```

##### Padding in the preprocessing phase

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

##### Parsing the padded message

After padding the coded message -from now on we will refer to it as **M**-, it will be parsed into **N** 512-bit blocks
```
M(1), M(2), M(3), ... M(N)
```

Each block is divided into 16 32-bit words
```
M(i)(0), M(i)(1), ... M(i)(15)
```
So the whole message M is composed of N blocks and each block is composed of 16 words.

#### Constants and Initial Hash Values

This section is not a phase of the hash function but to introduce some variables and constants that will be used in the last step of the algorithm which is the hash compuation.

##### Initial Hash Values

The sha-1 function in the end is composed of **five** concatinated **32-bit** variables like this
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

##### Constants Values

During the hash computations we're going to use **four** constants K(t). While t is the number of iteration in the hashing computation loop. The following are the constants in **hexadecimal** representaion

```
k(0 ≤ t ≤ 19) = 0x5a827999 
k(20 ≤ t ≤ 39 ) = 0x6ed9eba1 
k(40 ≤ t ≤ 59) = 0x8f1bbcdc  
k(60 ≤ t ≤ 79) = 0xca62c1d6   
```

**Note**: In the algorithm we need to convert the initial hash values and the constants from this hexadecimal format to their binary format.

#### Hash computations

This phase consists of generating a message scheduler (a function that helps us in extracting a word of the padded message) from the padded message and uses that scheduler with functions, constants and bitwise operations to iteratevly generate a series of hash values. Then the final hash will be the result of the hashing function which is basically called **Message Digest**.


##### Bitwise Operations

Before we get into message scheduling and the algorithm we need to know the bitwise operations we're going to use in this algorithm.

The bitwise operators we're going to use are the known ones `AND`, `NOT`, `OR` and `XOR`.
Adding to those there are other bitwise operators that we're going to use and they will be explained in this article since they are not very known amongst beginners.


**Bitwise right shift operator**

This bitwise operator has the notation of `x >> n` where **x** is the binary we're applying the operation on and **n** is the number of right shifts. Basically this operator shifts bits of **x** to the right by **n** positions like this:

```
x = 1101
x >> 1 = 0110
x >> 2 = 0011
```

**Bitwise left shift operator**

This bitwise operator is the reverse of the right shift operator it has the notation of `x << n` where **x** is the binary we're applying the operation on and **n** is the number of left shifts. Basically this operator shifts bits of **x** to the left by **n** positions like this:

```
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

```
x = 11001010
n = 1
ROTL (x , n) = 10010101
n = 2
ROTL (x, n) = 00101011
n = 3
ROTL (x, n) = 01010110
```

**Modulus Addition**

This operator makes an addition between binary numbers following this specific formula

```
(A+B) % 2^w
```

Where **A** and **B** are the decimal representation of two binary numbers. Later we encode the result into binary, and `0 < A < 2^w` and `0 < B < 2^w`. In other words **w** is the number of bits of **A** and **B**.

For SHA-1 in our case numbers are composed of **32 bits** so ` w = 32 `

```
(A+B) % 2^32
```


##### Message Scheduler
The message scheduler is a function like the following:

```
w(i)(t) = ( 0<= t <= 15)? M(i)(t) : ROTL( w(i)(t-3) XOR w(i)(t-8) XOR w(i)(t-14) XOR w(i)(t-16), 1 )
```


## Building the function in C

This section will be dedicated for building the whole function in C. So if you didn't understand some previous theoratical aspects like **Preprocessing**, **Hash computations** and **Message Scheduler** (I know you didn't 😁). This will be a practical example of everything mentionned before.

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

### Taking input

In this section we will focus on how the hash function in C will take input by discussing **the input format**, **message encoding** and **message decoding**.

### Input format

The input is going to be a **command line argument** inserted by the user, in this format
```
$ ./sha-1 -[sf] <input>
```

There will be two command line arguments, one a flag that indicates the input type and the input.
The input can be either a string like this:
```$ ./sha-1 -s "This is a string"```
or a file if we chose the f flag:
```$ ./sha-1 -f file_path```

#### If the input is a string

The string will be parsed and stored in a **string** data structure in C or to be more specific in an **array of characters** in C, each character will have its own array index like this:

```
$ ./sha-1 -s "This is a random text"
The C program will parse it into this : T = |T|h|i|s| |i|s| |a| |r|a|n|d|o|m| |t|e|x|t|
```

##### Implementation in C

