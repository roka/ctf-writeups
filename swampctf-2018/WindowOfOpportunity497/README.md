# SwampCTF 2018: Window of Opportunity
### Category: Reverse, Points 497, Solves 28

> While exploring an old military base, our party came across a old dusty terminal. We've tried to get access to the SYSTEM FLAG, but are unable to. Luckily, we managed to extract the OS binary, but the SYSTEM FLAG is encrypted! Without more understanding we'll just be guessing. Help us figure out how to get access and we'll split whatever we find with you.
> 
> Note: Brute force and you will be temporarily locked out from the SYSTEM FLAG.
> 
> Connect
> nc chal1.swampctf.com 1313
> 
> -=Created By: digitalcold=-



### Write-up
No user input is used for this challenge, if we connect to the challenge we get a token generated from seconds since epoch. 
```c
token =  (((time & 0xFFC) << 16) - 348403646)
  ^ ((unsigned char)(time & 0xF0) << 8)
  | ((time & 0xFFC) << 8) ` 
  | ((unsigned int)(time >> 8) << 24)
  | time & 0xFC;
```
The program will use the token as a pointer and attempt to jump to this address.

I solved the challenge by bruteforcing the token using python, after patching away the sleep() calls in OS.BIN with NOPs.
```python
import re
import datetime
import time
from subprocess import Popen, PIPE, STDOUT

notauth = re.compile("NOT AUTHORIZED")

while 1:
        p = Popen(['./OS.BIN_patched'], stdout=PIPE, stdin=PIPE, stderr=STDOUT)    
        osstdout = p.communicate()[0]

        if notauth.search(osstdout.decode()) is None:
            print datetime.datetime.now()

            m  = re.search(r'TOKEN: 0x(.*[0-9])', osstdout.decode())
            if m is not None:
                print str(datetime.datetime.now()) + " " + m.group(0)

        time.sleep(1)
```

There is three valid tokens 0xff7fcc46, 0xff3fcc46 and 0xff4fdc56, it possible to use them all to login however only the last one will give us the flag. Now we just need to wait until the correct time and connect to the server, when doing this we need to take into account that the server will sleep for 5 seconds before creating the token. I used a small c program for this:

```c
#include <stdio.h>
#include <sys/time.h>
#include <stdlib.h>

int main() {
  unsigned int i;
  struct timeval tv;

  unsigned int addr = 0xff4fdc56;
  unsigned int token = 0x0;

  while(token != addr) {
    gettimeofday(&tv, 0);
    i = tv.tv_sec + 5;

    token =  (((i & 0xFFC) << 16) - 348403646)
          ^ ((unsigned char)(i & 0xF0) << 8)
          | ((i & 0xFFC) << 8) |
          ((unsigned int)(i >> 8) << 24)
          | i & 0xFC;
  }

  system("nc chal1.swampctf.com 1313");

  return 0;
}

```

```
> LOGON
PROCESSING LOGON REQUEST .....

> GET TIME
THE TIME IS NOW 1522417429

> GENERATE TOKEN "1522417429"
GENERATING TOKEN .....
YOUR ACCESS TOKEN: 0xff4fdc56

> RUN "V:\OS\GETFLG.BIN" WITH KEY "0xff4fdc56"
ACCESS GRANTED
RUNNING ...
PERFORMING CHECKSUM ..... PASSED
LOADING DATA SECTION ..... LOADED

V:\OS\GETFLG.BIN > HELP
LIST
GET [ID]
SET [ID] [CONTENT]
CREATE [CONTENT]
EXIT

V:\OS\GETFLG.BIN > LIST
FLAG # |         DATE CREATED      |      LAST MODIFIED        |
-------+---------------------------+---------------------------+
1      | 2018-03-19T18:32:02+00:00 | 2018-03-29T04:56:40+00:00 |

V:\OS\GETFLG.BIN > GET 1
FLAG 1: flag{y0ur_t1m3_t0_sh1ne_is_N0W}

V:\OS\GETFLG.BIN > EXIT

> EXIT

TERMINATING CONNECTION...
```

> flag{y0ur_t1m3_t0_sh1ne_is_N0W}
