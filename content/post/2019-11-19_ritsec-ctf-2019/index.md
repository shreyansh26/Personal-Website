---
title: "RITSEC CTF 2019"
author: "Shreyansh Singh"
date: 2019-11-19T17:31:59.681Z
lastmod: 2019-11-29T21:04:58+05:30

description: ""

subtitle: "My writeups for RITSEC CTF 2019. A bit late, but I hope this helps someone!"

tags: [ctf, information security, infosec]
categories: [Information Security]

featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "RITSEC CTF 2019"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

A bit late for writeups, but still here are the solutions to the challenges I solved during the CTF. The CTF was from 15 Nov. 2019, 22:30 IST — Mon, 18 Nov. 2019, 10:30 IST. It was a decent CTF with quality challenges, from both beginner to advanced level.

**Update**: The scripts to solve and the flags are present in [this repo](https://github.com/wr47h/CTF-Writeups/tree/master/2019/RITSEC%20CTF%202019).


I'll do the writeups category-wise -

---

## Crypto  
&nbsp;  
**pre-legend — 100 pts**

`9EEADi^⁸:E9F3]4@&gt;⁴=2J32==^D@&gt;6E9:?8\FD67F=\C:ED64`

This is the provided cipher text. Since all of these are ASCII characters, we try a ROT of till say, 50.

{{< gist shreyansh26 e3386cba7303cdcb24b01d552b16aad4 >}}


On i=47, we get — https\x98\x8d\x8dgithub\x8ccom\x8dclayball\x8dsomething\x8buseful\x8britsec

There is a problem with the special characters, but we understand that is a GitHub repo, with the URL (after some testing) — [https://github.com/clayball/something-useful-ritsec](https://github.com/clayball/something-useful-ritsec).

Although there is nothing flag related in the repo, but the discord group of the CTF said that the link itself is the flag.

Flag —**RITSEC{https://github.com/clayball/something-useful-ritsec}**

&nbsp;  
**Shiny — 100 pts**

We are given the following text, and an image
`.‡8]5);483‡5;`


{{< figure src="https://cdn-images-1.medium.com/max/800/0*szkUp3DoP0hYUlyx" caption="gold-bug.jfif" >}}


This did not hit me directly, so I had to do a bit of Googling. I found that this is a reference to a short story by Edgar Allan Poe, called The Gold Bug which involves a substitution cipher. I found an [online tool](https://www.dcode.fr/gold-bug-poe) for the same.

This gives us the flag —**RITSEC{POEWASTHEGOAT}**

&nbsp;  
**random — 290 pts**

After connecting to _nc ctfchallenges.ritsec.club 8001_ we find that we are presented with a series of numbers and we have to guess the next. The challenge title tells us that we have something to do with the **random** function in the **C** language, because of the hint,
> Are you starting to 'C' a pattern?

We make a guess that whenever we request that host and port, the random function is initialized with a certain seed and we are given the first five random numbers generated from that seed. So, I wrote a simple C code to bruteforce all unix timestamps from 15th Nov 2019, 00:00 UTC to 17th Nov. 2019 00:00 UTC, and check for the seed. The code is shown below —

```cpp
#include <stdio.h>
#include <stdlib.h>

int main() {
    int start = 1573776000;  // 15th Nov 2019, 00:00 UTC
    int end = 1573948800;  // // 17th Nov 2019, 00:00 UTC
    for(int i=start; i<end; i++) {
        srand(i);
        int a = rand();
        int b = rand();
        int c = rand();
        int d = rand();
        int e = rand();
        int f = rand();
        if(a==1068399227 && b==161933545 && c==741438783 && d==1951874661 && e==1076387813) {
            printf("Seed: %d\n", i);
            printf("Next: %d\n", f);
            break;
        }
    }
    return 0;
}
```

We provide the next number, and get the flag — **RITSEC{404_RANDOMNESS_NOT_FOUND}**

---

## Misc
&nbsp;  
**Crack me If You Can — 391 pts**

In this challenge, after connecting to _nc ctfchallenges.ritsec.club 8080_, we find that we are presented with queries of hashes, and we have to break them in order to get the flag. They were NTLM and sha256 hashes. So, we used a combination of [crackstation.net](http://crackstation.net) and John the Ripper to crack both of them.

We found the flag — **RS{H@$HM31FY0UCAN}**

&nbsp;  
**Onion Layer Encoding — 100 pts**

The challenge says that the text is encoded using either Base16 or Baser32 or Base64 in a sequence. So we write a simple python script to solve it.

```python
import base64

flag = open("onionlayerencoding.txt","r").read()

while "RITSEC" not in str(flag):
    try:
        flag = base64.b16decode(flag)
    except:
        try:
            flag = base64.b32decode(flag)
        except:
            flag = base64.b64decode(flag)
            
print(flag)
```

The flag is — **RITSEC{0n1On_L4y3R}**

&nbsp;  
**AlPhAbEtIcAl Challenge - 100pts**

I couldn't solve this during the CTF, but saw other writeups and found that it was actually pretty interesting. The cipher text that is provided is —
> 59:87:57:51:85:80{:40:50:56:08:82:58:81:08:18:85:57:87:48:85:88:40:50:56:59:15:56:11:18:85:59:51:}

We see that the '{' and '}' are in place. So, this must represent the flag. The other numbers are assigned some alphabet starting from 'A'. After this we see that we have the following — ABCDEF{GHIJKLMJNECBOEPGHIAQIRNEAD}.

On this we use an online substitution solver like quipquip.com and also the fact that ABCDEF corresponds to RITSEC, we get the flag as — **RITSEC{YOUALPHABETIZEDYOURNUMBERS}**

---

## Web
&nbsp;  
**misdirection — 100 pts**

We are given a URL — [http://ctfchallenges.ritsec.club:5000/](http://ctfchallenges.ritsec.club:5000/) However, on clicking it we see that we are directed to another webpage [http://ctfchallenges.ritsec.club:5000/n](http://ctfchallenges.ritsec.club:5000/n) and the information that the webpage isn't redirecting properly. So, I decided to see what is happening, for that I did a simple _wget_ to the url.


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/3.png" caption="Running wget" >}}


We see that the page redirects to different pages and keeps doing that. We note that the last character is basically in the flag format when put together. We do that and get the flag — **RS{4!way5_Ke3p-m0v1ng}**

&nbsp;  
**Buckets of fun — 100 pts**

We are given the following URL — [http://bucketsoffun-ctf.s3-website-us-east-1.amazonaws.com/](http://bucketsoffun-ctf.s3-website-us-east-1.amazonaws.com/)

Taking a hint from the name of the challenge, we try the following URL in the browser —[http://bucketsoffun-ctf.s3.amazonaws.com](http://bucketsoffun-ctf.s3.amazonaws.com)


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/4.png" caption="The webpage" >}}


We see a file **youfoundme-asd897kjm.txt**

Heading to [http://bucketsoffun-ctf.s3.amazonaws.com/youfoundme-asd897kjm.txt](http://bucketsoffun-ctf.s3.amazonaws.com/youfoundme-asd897kjm.txt) we find the flag — **RITSEC{LIST_HIDDEN_FILES}**

---

## Forensics

&nbsp;  
**Take it to the Cleaners — 100 pts**

We are given an image


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/5.png" caption="The challenge" >}}


Performing basic recon, we check the metadata for the image using exiftool.


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/6.png" caption="exiftool output" >}}


In the user comment, we see a string which is probably base64 encoded.

Decoding it gives, **EVGFRP{SBERAFVPF_SNVYF_JBAG_URYC_LBH_URER}**

Looks rotated by an offset. We use [http://theblob.org/rot.cgi](http://theblob.org/rot.cgi) to get rotations by different offsets. This is ROT13 and the flag is — **RITSEC{FORENSICS_FAILS_WONT_HELP_YOU_HERE}**

&nbsp;  
**Long Gone — 100 pts**

We are provided a chromebin. Extract it as it is a tar archive.
> tar xzvf ./chromebin

We see there are a lot of folders, on inspecting the history we find it is an SQLite 3.X database. Loading it into DBBrowser, and inspecting the tables, shows an odd URL — us-central-1.ritsec.club/l/relaxfizzblur

Opening the url gives the flag — **RITSEC{SP00KY_BR0WS3R_H1ST0RY}**

---

## Pwn

&nbsp;  
**999 Bottles — 110 pts**

We are given 999 ELF files, each having a password as a single character. Basically, 999 crackmes with a one character password. If we check the disassembly of main function of any one —


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/7.png" caption="The disassembly for main" >}}


At 0x8048728 we see a comparison, where register edx (dl) stores our input character and eax (al) stores the value at address 0x804a039.

Also, in the disassembly, we have some character mappings to addresses —


{{< figure src="/post/2019-11-19_ritsec-ctf-2019/images/8.png" caption="Character mappings" >}}


So, one way to solve this challenge is to get the address to be compared and check the character at this address and automate it somehow.

During the CTF, however, I wrote a bruteforce script to try all characters for every ELF file.

```python
from pwn import *
import string

FOLDER = './elfs/'
filenames = []
s = string.digits + string.letters + string.punctuation

for i in range(1,1000):
    filenames.append(str(i).zfill(3) + '.c.out')flag = ''
f = open('flag.txt', 'w')
for file in filenames:
    for inp in s:
        p = process(FOLDER+file)
        p.recv()
        p.sendline(inp)
        a = p.recvline()
        if 'OK!' in a:
            flag += inp
            p.close()
            print("FLAG: " + flag)
            f.write("FLAG: " + flag)
            f.write('\n')
            break
        else:
            p.close()
            
f.write("FLAG: " + flag)
print(flag)
f.close()
```

Finally, we have the following string in the output generated -
> lr^wN${HnW&lt;DtVjk.RITSEC{AuT057v}^W!xT

Note the string in the flag format, that is the flag — **RITSEC{AuT057v}**

A better way to solve it actually using the method described above. The following script can help do that -

```python
import r2pipe  
import binascii  
import sys

for i in range(1, 1000):  
    print('elfs/{0:03}'.format(i))  
    b = r2pipe.open('elfs/{0:03}'.format(i) + '.c.out')

    disass = b.cmd('aaa; s main; pdd')  
    field = disass.split("eax = *(obj.")[1][0]  
    byte = disass.split(f'*(obj.{field}) = ')[-1][2:4]  
    print(binascii.unhexlify(byte).decode('ascii'), sep='')
```
   
This is all. Thanks for reading!

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).
