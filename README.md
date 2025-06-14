# ELF ARM Basic Crackme - Beginner Walkthrough

## Challenge Statement

> Get the validation password.  
> You are given an ARM 32-bit ELF binary (`ch23.bin`).  
> Your goal is to find the correct password to pass the check.

---

## Table of Contents

- [1. Preparation](#1-preparation)
- [2. Initial Static Analysis](#2-initial-static-analysis)
- [3. Dynamic Analysis (if possible)](#3-dynamic-analysis-if-possible)
- [4. Disassembly and Reverse Engineering](#4-disassembly-and-reverse-engineering)
- [5. Reconstructing the Password Logic](#5-reconstructing-the-password-logic)
- [6. Deriving the Solution](#6-deriving-the-solution)
- [7. Tips for Future CTFs](#7-tips-for-future-ctfs)

---

## 1. Preparation

Make sure you have the following tools:
- `file` (to identify binaries)
- `strings` (to extract printable strings)
- `radare2` or `Ghidra` (for disassembly)
- ARM emulation tools (`qemu-arm`, optional)

---

## 2. Initial Static Analysis

### 2.1. Identify the Binary

```sh
file ch23.bin
```

Output:
```
ch23.bin: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 2.6.26, BuildID[sha1]=..., stripped
```

### 2.2. Extract Strings

```sh
strings ch23.bin
```

Notable output:
```
Please input password
Checking %s for password...
Loser...
Success, you rocks!
```

These strings suggest the binary prompts for a password and prints success/failure.

---

## 3. Dynamic Analysis (if possible)

If you have an ARM environment or `qemu-arm` with the right libraries:

```sh
qemu-arm -L /usr/arm-linux-gnueabi/ ./ch23.bin
```

Enter some test passwords to see the output and behavior.

---

## 4. Disassembly and Reverse Engineering

### 4.1. Open in Radare2

```sh
r2 ch23.bin
```
Then run the analysis:
```
[0x000083b8]> aaa
[0x000083b8]> afl
```

Look for the main function, which is typically the largest:
```
0x00008470   17    484 main
```

### 4.2. Disassemble the Main Function

```
[0x000083b8]> s 0x00008470
[0x00008470]> pdf
```

#### Key Observations in the Disassembly

- The program checks that the input length is exactly **6**.
- Then, it performs several checks on the characters of the password using arithmetic and logical operations.

---

## 5. Reconstructing the Password Logic

Let the input be:  
`s[0] s[1] s[2] s[3] s[4] s[5]`

Checks observed:

1. `s[0] == s[5]`
2. `s[1] == s[0] + 1`
3. `s[3] + 1 == s[0]` (so `s[3] = s[0] - 1`)
4. `s[2] + 4 == s[5]` (so `s[2] = s[5] - 4`)
5. `s[4] + 2 == s[2]` (so `s[4] = s[2] - 2`)
6. Final check: `(s[3] ^ 0x72) == 0` (so `s[3] = 0x72` or ASCII `'r'`)

### Substitute Backwards

- From 6: `s[3] = 'r'`
- From 3: `s[0] = ord('r') + 1`
- From 2: `s[1] = s[0] + 1`
- From 4: `s[2] = s[0] - 4`
- From 5: `s[4] = s[2] - 2`
- From 1: `s[5] = s[0]`

Now fill in the values step by step using the ASCII numbers, but **do not write the final answer in the file**—let the reader derive it!

---

## 6. Deriving the Solution

**Your task:**  
- Work through the logic above, filling in the ASCII values (use `ord('r')` for `'r'`).
- Convert each result back to a character using `chr()`.
- Combine the six characters you get to form the password.

_This approach teaches you the method and lets you practice the final step!_

---

## 7. Tips for Future CTFs

- Always extract strings—sometimes the password is in plain sight!
- Look for the main function or the largest function in the binary.
- Track how user input is compared or manipulated.
- Reconstruct the logic using algebra if the relationships are mathematical.
- Don’t forget to check for off-by-one or overflow tricks with array indices.
- Practice makes perfect—try this style on other crackmes!

---

**Happy Hacking!**
