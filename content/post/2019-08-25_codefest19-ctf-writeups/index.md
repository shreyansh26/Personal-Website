---
title: "Codefest’19 CTF Writeups"
author: "Shreyansh Singh"
date: 2019-08-25T19:13:05.360Z
lastmod: 2019-11-29T21:04:46+05:30

description: ""

subtitle: "The Capture the Flag event for Codefest’19 was hosted from 8 pm, 23rd August 2019 to 12 noon, 24th August 2019 on Hackerrank."

tags: [ctf, information security, infosec, writeups]
categories: [Information Security]

featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Codefest 2019"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

The Capture the Flag event for [Codefest’19](http://codefest.tech) was hosted from 8 pm, 23rd August 2019 to 12 noon, 24th August 2019 on Hackerrank.

The contest link can be found [here](https://www.hackerrank.com/codefest19-ctf). There were a total of **1532** registrations and **518** people who were successful in solving atleast one challenge.

So, onto the writeups.

### **Welcome to Codefest 19! (Intro Challenge — 100pts)**

This was the introductory challenge. I had tried to make it a bit difficult than the normal introductory challenges, but I felt that it proved to be a bit difficult for the beginners.

{{< figure src="/post/2018-03-23_angstromctf-writeups/images/13.png" caption="The challenge" >}}


Here, first you had to join the telegram group linked in the proble. There you got the first half of the flag — **CodefestCTF{G3t_r3ady_**. For the other half there was a pinned message on the group.

> The other half of the flag was uploaded on the contest page yesterday by accident. It has now been removed. Can you find it?

For this you had to use [archive.org,](http://archive.org) there was a snapshot of the contest page created on 23rd Aug 2019. Viewing the [snapshot](https://web.archive.org/web/20190823133528/https://www.hackerrank.com/codefest19-ctf) got you the second half of the flag —**f0r_C0def3stCTF-8fb34fjr4bs43ur8}**.

So, the final flag is — **CodefestCTF{G3t_r3ady_f0r_C0def3stCTF-8fb34fjr4bs43ur8}**

&nbsp;  
### What language is this? (Misc — 100pts)

This was basically a esoteric language question. The given text was —

```
iiisdsiiioiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiodddddddddddoioiodoiiiiiiiiiiiiiioiodddddddddddddddddddddddddddddddddddddddddddddddddoiiiiiiiiiiiiiiiiioddddddddddddddoiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiioddddddddddddddddddddddddddddddddddddoiiiiiiiiiiiiiioiiiiiiiodddddddddodddddddddddddddddddddddddddddddddddddddddddddddddddoddddddddddddddddddddddddddddddddddddddsiiiiiiiiioddddddddoddddddoiiiiiiiiiiiiiiiiiiiiioddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddoddddddddddddddddddddddddddddddddsiiisisdddddoddddddddddddddddddddddddddddodddddddddddddddddddoddddddddddddddddddddddddddddddddsiiisisoioiodoiiiiiiiiiiiiiioiodddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddddoiiiiiiiioddddddddddddddddddddddddddddddddddddddddddddddsiiiio
```

The language was [Deadfish](https://esolangs.org/wiki/Deadfish). You could use an online decoder for that language, something like [this](https://www.dcode.fr/deadfish-language).

The final flag was — **CodefestCTF{Welc0me_t0_C0defest19}**

&nbsp;  
### Gibberish file (Misc — 100pts)

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/2.png" caption="The challenge" >}}



The hint was in the problem statement. You had to reverse the file to find the flag. A simple one-line script could do it

```python
open("output2.txt", "wb").write(open("output.txt", "rb").read()[::-1])
```


The resulting had some text like

> 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ 𝝩𝗵𝙚 𝗳𝒍𝙖𝗴 𝒊𝙨 𝐋𝒊𝐓𝚬𝐫𝚨𝐋﹏𝕽𝜠𝓥Ｅℜ𝕊𝐢𝙣𝓖ꓸ

The flag was the ASCII analog of each unicode character.

The flag was — **CodefestCTF{LiTErAL_REVERSinG}**

&nbsp;  
### Image Corruption (Forensics — 100pts)

In the challenge, you were given a link to a corrupted _.bmp_ [file](https://drive.google.com/file/d/1t5d_lKkdoG1aicBJYhM8wqh7Ispk0G4U/view). On viewing the file in a hex editor, and also checking the magic bytes —


{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/3.png" caption="Hex view of the image" >}}

We know there is something to do with “matrix”. Also for a normal _.bmp_ file the initial magic bytes are 424d 8a44 1300. XORing this with the first six bytes of the given file also gives you “matrix”. So to solve the challenge, we XOR the whole image with “matrix”.

{{< gist shreyansh26 391d4103c8175cd3484c286d7c51dfd7 >}}

Run the script, and you obtain the correct file.

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/4.jpeg" caption="The correct file" >}}


The flag is — **CodefestCTF{f1l35_h4v3_m461c_by735}**

&nbsp;  
### Mail capture (Steganography— 100pts)

You are presented with a “email friendly text”. This was encoded to unicode by a tool called **uuencode**. It can be decoded by using **uudecode**, a decoder for such formats. Running **uudecode** with the file gives an output file called “flag_encoded”. The contents are the flag — **CodefestCTF{7h15_15_4_c001_3nc0d1n9}**

&nbsp;  
### Cats are innocent, right? (Steganography— 500pts)

This challenge was based on LSB steganography. I had used a tool called [**stegify**](https://github.com/DimitarPetrov/stegify).

The challenge image -


{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/5.jpeg" caption="Challenge image" >}}


On running the command -
> stegify -op decode -carrier cute_kittens.jpg -result hello

We get a _hello.zip_ file which was embedded in the LSBs of the image. The zip file had a file inside it but that was of no use. The flag was appended at the end of the zip file.

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/6.png" >}}

The flag is appended at the end of the zip file


The flag is — **CodefestCTF{h1d1ng_b3h1nd_1nn0c3nt_k1tt3n5}**

&nbsp;  
### Weird encoding (Misc— 200pts)


{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/7.png" caption="The challenge" >}}


We are given the following “encoding”

```
0x85+1x1+0x14
0x7+1x1+0x7+1x1+0x9+1x2+0x3+1x4+0x3+1x1+0x6+1x5+0x1+1x1+0x2+1x1+0x1+1x2+0x13+1x2+0x3+1x1+0x8+1x1+0x5+1x2+0x8
0x1+1x5+0x18+1x3+0x3+1x1+0x16+1x2+0x1+1x1+0x5+1x2+0x2+1x1+0x3+1x1+0x4+1x2+0x3+1x3+0x3+1x1+0x2+1x2+0x4+1x3+0x8
0x3+1x1+0x7+1x1+0x11+1x2+0x1+1x1+0x3+1x5+0x12+1x1+0x2+1x1+0x7+1x1+0x10+1x1+0x3+1x2+0x1+1x1+0x5+1x3+0x4+1x1+0x1+1x2+0x2+1x1+0x4
0x3+1x1+0x3+1x1+0x7+1x2+0x3+1x1+0x2+1x1+0x2+1x1+0x7+1x1+0x11+1x2+0x2+1x2+0x5+1x2+0x10+1x1+0x3+1x1+0x2+1x1+0x3+1x2+0x2+1x1+0x4+1x4+0x7
0x3+1x1+0x3+1x1+0x3+1x1+0x1+1x3+0x10+1x1+0x7+1x1+0x7+1x1+0x3+1x1+0x3+1x1+0x1+1x2+0x2+1x3+0x8+1x5+0x4+1x1+0x3+1x9+0x1+1x3+0x7
0x3+1x1+0x3+1x3+0x1+1x1+0x1+1x4+0x9+1x1+0x6+1x2+0x2+1x1+0x7+1x2+0x3+1x1+0x2+1x1+0x4+1x1+0x10+1x1+0x6+1x1+0x7+1x1+0x7+1x4+0x4
0x5+1x1+0x1+1x1+0x1+1x1+0x1+1x1+0x4+1x2+0x7+1x2+0x3+1x4+0x11+1x1+0x4+1x1+0x2+1x1+0x3+1x2+0x6+1x1+0x3+1x1+0x6+1x1+0x7+1x1+0x1+1x1+0x1+1x5+0x7
0x7+1x1+0x1+1x1+0x1+1x1+0x2+1x3+0x7+1x5+0x16+1x1+0x4+1x1+0x2+1x1+0x1+1x3+0x3+1x6+0x2+1x1+0x2+1x1+0x1+1x5+0x5+1x1+0x2+1x1+0x4+1x1+0x7
0x18+1x5+0x13+1x6+0x27+1x1+0x14+1x1+0x2+1x2+0x2+1x1+0x5+1x1+0x2
0x1+1x1+0x5+1x1+0x4+1x1+0x3+1x1+0x8+1x1+0x8+1x1+0x9+1x1+0x8+1x1+0x5+1x1+0x17+1x1+0x10+1x3+0x9
0x68+1x1+0x11+1x1+0x19

```


Here a bit of observation was required to figure out that the “x” symbol mean concatenation _n_ number of a character, like 0x5 will mean 00000. And “+” would mean concatenation of two strings of different type. Also, one will also have to decide on 0 representing 255 255 255 i.e. the color _white_ and 1 representing 0 0 0 , i.e. the color _black_. You could have experimented with both combinations but eventually you would get the correct mapping.

The following script can help generate the image.

{{< gist shreyansh26 9f262ca29ec6bcb5c8b66e1feb95cf6e >}}

The obtained image is this -

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/8.png" caption="You may want to zoom in a bit">}}


The flag is — **CodefestCTF{This_15_7h3_f14g}**.

&nbsp;  
### Linux RE 1 (Reversing — 300pts)

This challenge was a bit difficult to solve using a debugger due to some anti-debugging techniques that were implemented. Also, initially the ELF was packed using UPX, which was visible as a string when you would have run the **strings** command. So, first use
> upx -d

with the ELF to decompress it.

For the next part, You could use a disassembler or a decompiler to get the source code and eventually reverse the binary. The executable was generated from a C++ file hence it was a bit messy to view in a decompiler.

The decompiled view (using Ghidra) of the main function (the interesting part) is the following -


{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/9.png" caption="Decompiled main function" >}}


The _key_int_ and _enc_int_ are global variables. The main logic of the ELF is in the **rahasya** function.

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/10.png" caption="Decompiled rahasya function" >}}


This basically takes two strings and XORs them and returns the XORd string. The two strings it takes as input are the user input and the _key\_int_ string. The XORd data is matched with the _enc\_int_ data.

So, basically to reverse the binary you have to XOR both the _key_int_ and _enc_int_ data.


{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/11.png" caption="enc_int data" >}}

{{< figure src="/post/2019-08-25_codefest19-ctf-writeups/images/12.png" caption="key_int data" >}}


Basically,
> int _enc\_int[]_ = {80, 93, 3, 67, 3, 86, 11, 110, 64, 2, 90, 27, 84, 28, 110, 75, 3, 69, 52, 6, 11, 5, 80, 88, 90, 88};

> int _key\_int[]_ = {49, 51, 51, 55, 107, 101, 121};

XOR both of them, and you get _an0th3r_s1mp1e_x0r_cr4ckm3_

So, the flag is **CodefestCTF{an0th3r_s1mp1e_x0r_cr4ckm3}**

&nbsp;  
### Linux RE 2 (Reversing — 500pts)

Again we open the file in IDA or any disassembler and/or decompiler we see that the input should satisfy a set of conditions on the letters of the input.

The conditions can be translated as

{{< gist shreyansh26 c0c225fa5daa05da85bb0534dc5438a9 >}}

We can use some kind of SMT solver like z3 to find the password.

{{< gist shreyansh26 bccc43909e68ddd7d219f59fc48da4c5 >}}

The obtained password is — _shouldve_used_some_tool_

The flag, hence, is **CodefestCTF{shouldve_used_some_tool}**

&nbsp;  
### Windows RE (Reversing — 500pts)

In this problem, the Windows exe file (actually a .NET file) that was provided, was packed with [ConfuserEx](https://yck1509.github.io/ConfuserEx/). We can use [NoFuserEx,](https://github.com/CodeShark-Dev/NoFuserEx) which is a free deobfuscator for this packer.

Then, open the executable in any .NET decompiler like dnSpy and check the **Form** function to get the password as well as the flag.
> Password — thisisa1337password

The flag — **CodefestCTF{51mp13_1npu7_v411d4710n_8u7_w17h_4_7w157}**

&nbsp;  
### No Fatshaming (Web — 600pts)

I’ll cheat a bit here xD. You can read my friend Yashit’s [awesome writeup](https://medium.com/@yashitmaheshwary/no-fatshaming-web-challenge-writeup-codefest19-ctf-1deea5a2ea49) on the challenge.

Flag is — **CodefestCTF{1AmTeHHHAX00Rr4uj8rfi4e$%y5yhrf}**

_Hope you had a great time solving the challenges and that it was a good learning experience for beginners._

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).
