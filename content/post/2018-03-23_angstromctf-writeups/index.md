---
title: "AngstromCTF Writeups"
author: "Shreyansh Singh"
date: 2018-03-23T12:37:04.861Z
lastmod: 2019-11-29T21:04:24+05:30

description: ""

subtitle: "These are the writeups to the problems I solved during the AngstromCTF."


tags: [ctf, information security, infosec, writeups]
categories: [Information Security]

featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Angstrom CTF"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

These are the writeups to the problems I solved during the AngstromCTF.

---

## MISC
&nbsp;  
**1. Waldo1**

We are given a zip file — flags.zip containing flags of countries. The file flag5.png, we see on opening has the flag.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/2.png" caption="Flag-Waldo1" >}}


&nbsp;  
**2. Waldo2**

In this problem, we are given multiple flag images in a folder. Judging by the problem, it seems that one image is different. We see the md5 hash of the few files which are the same-**9f6e902c233020026caf0ebbb1cf0ff5**. So we write the following script-

{{< gist shreyansh26 06c29b45da61157827bb20a355faa6c9 >}}


So, the filename we get is **waldo339.jpg**. Running `strings` on the file we get the flag as — **actf{r3d_4nd_wh1t3_str1p3s}**.

&nbsp;  
**3. That’s not my name**

We are given a pdf file — gettysburg.pdf, but on trying to open it, it does not open, giving incorrect file format error. We run `binwalk` on the file to see that it infact is a `docx` file. We change the extension to `.docx` anf on opening we get the flag as — **actf{thanks_mr_lincoln_but_who_even_uses_word_anymore}**.

&nbsp;  
**4. File Transfer**


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/3.png" caption="Capture" >}}


The highlighted packet shows a JPEG image capture. We export the JPEG as bytes to get the image.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/4.jpeg" caption="Flag — File Transfer" >}}

&nbsp;  
**5. GIF**

On running `binwalk` on the given image, we see that it is infact a collection of many images.

So , we run the command `binwalk -D 'png image:png' jiggs.gif.png`. On inspecting the extracted files, we see an image which has the flag.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/5.png" caption="Flag — Gif" >}}

---

## Crypto
&nbsp;  
**1. Warmup**

From the term **_fine_** cipher, we get the hint that it could be an **Affine cipher**. We use an online Affine cipher [solver](https://www.dcode.fr/affine-cipher) to get the flag as — **actf{it_begins}**.

&nbsp;  
**2. Back to Base-ics**

We are given the following cipher text -


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/6.png" caption="Ciphertext" >}}


Now we can easily see that the Part 1 is binary(base 2) and Part 3 is hexadecimal(base 16). On decoding them using any online converter, we get

Part 1: **actf{0ne_tw0_f0**

Part 3: **n_th1rtytw0_s1x**

Also judging from the title of the problem, we can say that all the ciphers have the base of some power of two. We guess that Part 2 could be base 8(octal). using an online octal to text converter we get,

Part 2: **ur_eight_sixt33**

The last one looks like base64. On decrypting, we get

Part 4: **tyf0ur_no_m0re}**

So, the flag is — **actf{0ne_tw0_f0ur_eight_sixt33n_th1rtytw0_s1xtyf0ur_no_m0re}**

&nbsp;  
**3. XOR**

This looks like a singlebyteXOR problem. We use the following script


{{< gist shreyansh26 982f2b6974fec2195f752bb86cc91393 >}}

On seeing all the plain texts, we get the flag as — **actf{hope_you_used_a_script}**.

&nbsp;  
**4. Intro to RSA**

This is a classical RSA problem, we use the following script to decrypt


{{< gist shreyansh26 f6af59e02f2db1d0e27e88b5b8084585 >}}


So the flag is — **actf{rsa_is_reallllly_fun!!!!!!}**.

---

## WEB
&nbsp;  
**1. Source Me 1**

Here, we are presented with a login page. On inspecting the source, we find the password —**f7s0jkl**, in the comments. **** So, we login with the username as `admin` and password as `f7s0jkl`.

This gives us the flag-**actf{source_aint_secure}**.

&nbsp;  
**2. Get Me**

Initially all we have is a button with the message that only authorized users are allowed to pass. On clicking the button, we get the message that we are not authorized. However in the url bar we see that the get parameter is `auth=false`. We change it to `auth=true`and hit enter.

We then get the flag — **actf{why_did_you_get_me}**.

&nbsp;  
**3. Sequel**

This is a classic case of SQL injection(SQLi). The hint here is the name of the problem which is pronunciation of SQL.

We enter both username and password as `'or''='`.

This gives us the flag — **actf{sql_injection_more_like_prequel_injection}**.

&nbsp;  
**4. Source Me 2**

We are give another login page. Here, too, the username is `admin`. On inspecting the source, we find the script which converts our entered password to md5 and compares it to the hash **bdc87b9c894da5168059e00ebffb9077**. We use an [online md5 decryptor to](http://www.md5online.org/) get the password as `password1234`. Entering this gives the flag — **actf{md5_hash_browns_and_pasta_sauce}**.

&nbsp;  
**5. Madlibs**

Here, from the Flask code we see that there is a variable app.secret_key, which is basically a config variable. So we head to Tale of a Person section and enter `{{config}}` as the Author name and any random strings in the other options.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/7.png" >}}


Here we see the SECRET_KEY variable assigned to the flag, **actf{wow_ur_a_jinja_ninja}**

---

## Reversing(RE)
&nbsp;  
**1. Rev1**

First, we run `strings` on the given ELF executable. We see the string, **s3cret_pa55word**. This could be the secret password the program is looking for. On running the executable and giving the above string as key, we get the flag. This is to be done on the shell server.

&nbsp;  
**2. Rev2**

The ELF on executing asks for a number to be guessed. We use radare2 to disassemble the code.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/8.png" >}}

{{< figure src="/post/2018-03-23_angstromctf-writeups/images/9.png" >}}


The highlighted hex, 0x11d7 is **4567** in decimal. On entering this, the program now asks us to give two two-digit numbers. We again analyze the disassembled code.

{{< figure src="/post/2018-03-23_angstromctf-writeups/images/10.png" >}}


This tells us that the product of the two numbers should be 0xd67 i.e **3431**. From [this link,](http://www.mathwarehouse.com/arithmetic/numbers/prime-number/prime-factorization.php?number=3431) we find that the numbers are **47** and **73**. We enter them in ascending order, i.e 47 and then 73.

We get the flag as — **actf{4567_47_73}**.

---

## Binary
&nbsp;  
**1. Accumulator**

Here the ideas is to keep adding integers to an `int` variiable and without explicitly entering negative values, we have to make the result negative. This can be done by integer overflow.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/11.png" >}}


Running these inputs on the shell server will give us the flag.

&nbsp;  
**2. Cookie Jar**

This is a buffer overflow problem. Although we never explicitly gave a value to numCookie, we can overflow the buffer so that it gets a value. I fwe the following input — aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa99999998 to the program, we get the flag.

The flag is — **actf{eat_cookies_get_buffer}**.

&nbsp;  
**3. Number Guess**

We take the help of the hint given. The most common vulnerability of the printf function is the use(or not) of format strings.

In the code, just before the `printf(buf)` the two random integers are initialized. So, when we are asked for our name, if we give the following input, **%d %d %d %d %d %d %d %d %d %d %d %d** . This would give us the other numbers in the stack. On running this, we take the 3rd and the 9th value as rand1 and rand2. We add them and give the result as our guess.


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/12.png" >}}


So the flag is -**actf{format_stringz_are_pre77y_sc4ry}**.

&nbsp;  
**4. Rop to the Top**

This is an example of Return Oriented Programming (ROP) vulnerability which is basically buffer overflow to access the non-executable stack. To exploit it we can use the following set of commands-


{{< figure src="/post/2018-03-23_angstromctf-writeups/images/13.png" >}}


We find that the address of **the_top** function is **0x8048db**. Also the buffer size is **0x28**.

So, the following command works for us-

**./rop_to_the_top32 "$(python -c 'print "A"\*0x28 + "BBBB" + "\xdb\x84\x04\x08"')"**

We enter the character ‘A’ to fill the size of the buffer, “BBBB” to replace the current stack pointer (%ebx) followed by the address to which we wish to point to, here the address of **the_top** function.

Running the above command on the shell server gives us the flag.

---

_For more writeups, you can follow me on_ [_Github_](https://github.com/wr47h)
