---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Androids Encryption (Crypto) - Pwn2Win CTF 2020"
subtitle: "My writeup for Androids Encryption challenge in the Pwn2Win CTF 2020"
authors: ["Shreyansh Singh"]
tags: [crypto, infosec, writeups]
categories: [Information Security]
date: 2020-06-01T12:32:46+05:30
lastmod: 2020-06-01T12:32:46+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

### crypto 115 - 108 solves

> We intercept an algorithm that is used among Androids. There are many hidden variables. Is it possible to recover the message?

> Author: andre_smaira

> Server: nc encryption.pwn2.win 1337

[Challenge link](https://pwn2.win/NIZKCTF-js/challenges/androids_encryption)  
[Challenge files](/post/2020-06-01_androids_encryption-pwn2win-2020/files/server.py)

On connecting to the challenge service, we are given two options - 

{{< figure src="/post/2020-06-01_androids_encryption-pwn2win-2020/images/options.PNG" >}}

Also, in the server.py file, we see there are two functions, `enc_plaintext` and `enc_flag`. Both these functions call the `encrypt` function. 

The `enc_plaintext` functions calls `encrypt` with the plaintext supplied by the user (encoded in base64), key1 and iv1 (which are secret values) as arguments.  
The `enc_flag` fucntion takes as arguments the secret flag and iv2 and key2 which are actually derived from iv1, key1 and the flag.

```python
iv2 = AES.new(key1, AES.MODE_ECB).decrypt(iv1)
key2 = xor(to_blocks(flag))
```

The `encrypt` function returns the base64 encoding of the (iv+ciphertext) string.

### Exploit

We see that every time `encrypt` is called, the key2 and iv2 values are updated. iv2 becomes the AES-ECB decryption of the old iv2 and key2 is now the first block of the ciphertext (since the xor function with one argument simply returns the first block of the argument). So, our goal is now to recover the key2 and iv2 values so we can then reverse the `encrypt_flag` function and recover the flag.

{{< figure src="/post/2020-06-01_androids_encryption-pwn2win-2020/images/update.PNG" >}}

If we call `encrypt_flag` (Choice 2) the first time, then the new key2 value will be the first block of the ciphertext. Nnow during this, the iv2 has also been updated but we don't know that value, since the first part of the returned value, is the old iv2. 

So, what we do next is call the `encrypt_flag` (Choice 2) function again, then we get the new iv2 value along with the ciphertext. This means taht now we know the iv2 and the key2 value taht was used to encrypt the flag to obtain the ciphertext. What remains now, is just to reverse the `encrypt` function and call it with our values of the ciphertext, key2 and iv2. This will get us the flag.

The `decrypt` function can be written as -

```python
assert len(key) == BLOCK_SIZE, f'Invalid key size'
    assert len(iv) == BLOCK_SIZE, 'Invalid IV size'
    assert len(txt) % BLOCK_SIZE == 0, 'Invalid plaintext size'
    bs = len(key)
    blocks = to_blocks(txt)
    ctxt = b''
    aes = AES.new(key, AES.MODE_ECB)
    curr = iv
    for block in blocks:
        ctxt += xor(curr, aes.decrypt(block)) # Inverse of the encrypt function
        curr = xor(ctxt[-bs:], block)
    return ctxt
```

The complete exploit code is shown below - 

```python
from pwn import *
import base64
from Crypto.Cipher import AES

p = remote("encryption.pwn2.win", 1337)
# context.log_level = 'debug'

BUFF = 256
BLOCK_SIZE = 16
key2 = None
iv2 = None

def to_blocks(txt):
    return [txt[i*BLOCK_SIZE:(i+1)*BLOCK_SIZE] for i in range(len(txt)//BLOCK_SIZE)]

def xor(b1, b2=None):
    if isinstance(b1, list) and b2 is None:
        assert len(set([len(b) for b in b1])) == 1, 'xor() - Invalid input size'
        assert all([isinstance(b, bytes) for b in b1]), 'xor() - Invalid input type'
        x = [len(b) for b in b1][0]*b'\x00'
        for b in b1:
            x = xor(x, b)
        return x
    assert isinstance(b1, bytes) and isinstance(b2, bytes), 'xor() - Invalid input type'
    return bytes([a ^ b for a, b in zip(b1, b2)])

def decrypt(txt, key, iv):
    assert len(key) == BLOCK_SIZE, f'Invalid key size'
    assert len(iv) == BLOCK_SIZE, 'Invalid IV size'
    assert len(txt) % BLOCK_SIZE == 0, 'Invalid plaintext size'
    bs = len(key)
    blocks = to_blocks(txt)
    ctxt = b''
    aes = AES.new(key, AES.MODE_ECB)
    curr = iv
    for block in blocks:
        ctxt += xor(curr, aes.decrypt(block)) # Inverse of the encrypt function
        curr = xor(ctxt[-bs:], block)
    return ctxt

def encrypt_flag(p):
	p.recvuntil(b"Choice: ")
	p.sendline(b"2")
	x = p.recvline().strip()
	y = base64.b64decode(x)
	return y[:16], y[16:]

def enc_plaintext(p, plaintext):
	p.recvuntil(b"Choice: ")
	p.sendline(b"1")
	p.recvuntil(b"Plaintext: ")
	p.sendline(plaintext)
	x = p.recvline().strip()
	y = base64.b64decode(x)
	return y[:16], y[16:]

def getDecoding(s):
	y = base64.b64decode(s)
	return y[:16], y[16:]

iv2_orig, flag_enc_orig = encrypt_flag(p)
print(iv2_orig)
print(flag_enc_orig)

key2_new = xor(to_blocks(flag_enc_orig))
print(b"Key2 :" + key2_new)
iv2_new, flag_enc_cipher_new = encrypt_flag(p)
print(b"iv2 :", iv2_new)
print(b"flag_cipher_new :", flag_enc_cipher_new)

print(decrypt(flag_enc_cipher_new, key2_new, iv2_new).decode())
```

Running this, prints the flag - **CTF-BR{kn3W_7h4T_7hEr3_4r3_Pc8C_r3pe471ti0ns?!?}**

And yeah, after reading the flag, I realised it was actually AES in PCBC mode.

---

<script type="text/javascript" src="//downloads.mailchimp.com/js/signup-forms/popup/unique-methods/embed.js" data-dojo-config="usePlainJson: true, isDebug: false"></script>

<!-- <button style="background-color: #70ab17; color: #1770AB" id="openpopup">Subscribe to my posts!</button> -->
<div class="button_cont" align="center"><button id="openpopup" class="example_a">Subscribe to my posts!</button></div>

<style>
    .example_a {
        color: #fff !important;
        text-transform: uppercase;
        text-decoration: none;
        background: #3f51b5;
        padding: 20px;
        border-radius: 5px;
        cursor: pointer;
        display: inline-block;
        border: none;
        transition: all 0.4s ease 0s;
    }

    .example_a:hover {
        background: #434343;
        letter-spacing: 1px;
        -webkit-box-shadow: 0px 5px 40px -10px rgba(0,0,0,0.57);
        -moz-box-shadow: 0px 5px 40px -10px rgba(0,0,0,0.57);
        box-shadow: 5px 40px -10px rgba(0,0,0,0.57);
        transition: all 0.4s ease 0s;
    }
</style>


<script type="text/javascript">

function showMailingPopUp() {
    window.dojoRequire(["mojo/signup-forms/Loader"], function(L) { L.start({"baseUrl":"mc.us4.list-manage.com","uuid":"0b10ac14f50d7f4e7d11cf26a","lid":"667a1bb3da","uniqueMethods":true}) })

    document.cookie = "MCPopupClosed=;path=/;expires=Thu, 01 Jan 1970 00:00:00 UTC";
}

document.getElementById("openpopup").onclick = function() {showMailingPopUp()};

</script>

&nbsp;  

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).