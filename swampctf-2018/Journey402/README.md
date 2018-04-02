# SwampCTF 2018: Journey
### Category: Reverse, Points 402, Solves 150

> You have reached a side challenge in your quest, and you must find a way to reach the password. If you do, you can save your whole team, if your dungeon master allows. However, there are a few perfect 20s you could role to get there quicker than expected. Don't get distracted, and jump through each stage to complete the quest. No tricky business!
> 
> -=Created By: noopnoop1=-


### Write-up
After running the challenge it will ask for a password:
```
$ ./journey 
As you play this game, there will be many adventures for you to take, quests on the side of this great journey.
This is one of the quests here, and you may not know it yet. You will know when you complete the test, because all of > the die will have been cast in your favor.
Prove your worth, enter a password to continue!
```


When opening journey in binary ninja only four subroutines are found and the _start subroutine is just looping, this suggests the binary is packed/encrypted. Runnings strings on the binary lets us know it was packed using upx, which is a common packer.

> $Info: This file is packed with the UPX executable packer http://upx.sf.net $
> $Id: UPX 3.91 Copyright (C) 1996-2013 the UPX Team. All Rights Reserved. $

The binary can be unpacked with the tool from http://upx.sf.net 

The program will loop trough each character of the password and subtract it by 14231234143212433 (mod 10), after usage
the last digit will be removed. This new string is compared against the value "theresanotherstep".

```c
  scanf("%17s", password);
  len = strlen(password);
  n = 14231234143212433;
  
  for ( i = 0; i < len; ++i ) {
    tmp = n % 10;
    n /= 10;
    password[i] -= tmp;
  }
  
  if ( !strcmp(password, "theresanotherstep") ) {
    // SUCCESS
  }
```
I used python to get the inverse of the above method. 
```python
n = 14231234143212433
decoded = "theresanotherstep"
passwd = ""

for ch in decoded:
    passwd += chr(ord(ch) + (n % 10))
    n = int(n / 10)

print passwd
```

```
$ ./journey 
As you play this game, there will be many adventures for you to take, quests on the side of this great journey.
This is one of the quests here, and you may not know it yet. You will know when you complete the test, because all of the die will have been cast in your favor.
Prove your worth, enter a password to continue!
wkitfudrpxkgsvviq
Congratulations, you have reached the next stage in the mission. As close as you are, you can almost smell the perfect role!
However, no tricky business is allowed. It's about the journey, not the destination, so the password you entered does matter.
In fact, the password is the answer! Submit in the form flag{...}
```

> flag{wkitfudrpxkgsvviq}
