---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "hxp 36C3 CTF Writeups"
subtitle: "The writeups for the challenges I solved in the first MAJOR CTF that I participated in after a long time."
authors: ["Shreyansh Singh"]

tags: [rev, wasm, android, misc, 36C3, information security, infosec, writeups]
categories: [Information Security]

date: 2019-12-29T14:06:46+05:30
lastmod: 2019-12-29T14:06:46+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "hxp 36C3 CTF"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

The hxp CTF happens every year along with the Chaos Communication Congress (a top security conference). This year was the 36th edition. This CTF is a major CTF, you know this when the CTF has a rating weight of 63.0 on CTFTime. Also, it is one of the qualifier events of [DEFCON 2020 CTF](https://www.oooverflow.io/dc-ctf-2020-quals/).

I was playing solo on this one and gave one day to this CTF. I managed to solve 2 problems in the main CTF and 2 in the [Junior CTF](https://kuchenblech.xyz/).  

Here are the writeups for the challenges I solved.

---

# Main CTF

## 1337 Skills - Android, Rev

> App: [Link](https://play.google.com/store/apps/details?id=com.progressio.wildskills)  
> Connection: nc 88.198.154.132 7002

First, I installed the app on my phone, to try to play around with it a bit. But the very first page was a login type screen asking for a code. I knew I had to open it in a decompiler to see what is happening and figure out the code. I extracted the APK of the app and opened it up in jadx.

First I took a look at the AndroidManifest.xml, to find the launcher activity.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/manifest.png" >}}

The class we have to check out first is the `com.progressio.wildskills.MainActivity`. Opening this we see that the `onCreate` method calls the `activateApp` method to check the activation code.

```java
public void activateApp(View view) {
    int i;
    try {
        i = Integer.parseInt(this.editTextActivation.getText().toString());
    } catch (NumberFormatException unused) {
        i = -1;
    }
    Calendar instance = Calendar.getInstance();
    if (i == ((int) (Math.pow((double) (instance.get(3) * instance.get(1)), 2.0d) % 999983.0d))) {
        findViewById(R.id.scrollViewActivation).setVisibility(4);
        ((InputMethodManager) getSystemService("input_method")).hideSoftInputFromWindow(this.editTextActivation.getWindowToken(), 0);
        SharedPreferences.Editor edit = this.prefsmain.edit();
        edit.putBoolean("Activated", true);
        long time = new Date().getTime();
        edit.putLong("Installed", time);
        edit.putLong("ActivationDate", time);
        edit.commit();
        return;
    }
    Toast.makeText(this, "Ungültiger Aktivierungscode", 1).show();
    this.editTextActivation.requestFocus();
    ((InputMethodManager) getSystemService("input_method")).showSoftInput(this.editTextActivation, 1);
}
```

We have to pay attenton to 

```java
i = ((int) (Math.pow((double) (instance.get(3) * instance.get(1)), 2.0d) % 999983.0d))
```

For 29th December 2019, this value is a constant and equal to `76429`. Entering this, we get access to the app. Next on the top right corner of the app, there are options namely Sales, Leadership, Smart Profuction (the current page) and Service Roadmap. Each of these (except Smart Production) require their own activation codes. We deg deeper into the app's code for this.

One thing I note is that on entering a wrong code, the following message is shown as a Toast - "Ungültiger Aktivierungscode". So, I used Jadx's Text Search to find all instances of this. We find this

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/codes.png" >}}

These are basically the codes for the three sections. Now all we have to do is connect to the given server and port and answer with these codes.

```
Activation code: 
76429
activated!
Sales activation code: 
sgk258
activated!
Leadership activation code: 
wmt275
activated
Service Roadmap (SRM) activation code: 
udh736
activated!
```

After this, we get the flag - `hxp{thx_f0r_4773nd1n6_70d4y}`

&nbsp;  

## xmas_future - Rev


Files: [files.zip](/post/2019-12-30_hxp-36c3-ctf/files/files.zip)

This challenge is really close to my heart because this was the FIRST time ever I solved a WASM reveresing challenge. I literally had no clue on how to proceed, did a bit of researching and finally worked it out.

First I thought of converting the .wasm file into some readable code like in C. I used the official [WebAssembly binary toolkit (wabt)](https://github.com/WebAssembly/wabt) for this. I used both the wasm2c and wasm2wat to get readable code. In the C file, there was one interesting function which was being called from the hxp2019.js file, the `check` function, specifically the `$hxp2019::check::h578f31d490e10a31` fnction. But it was a lot of code and I couldn't make anyting out of it. Then I decided to read few wasm related CTF writeups. I learnt that I could actually use the debugger in the Chrome DevTools to go through it.

Opening the html file directly in the browser wasn't loading the js file due to CORS. I copied the folder into my `/var/www/html` folder and accessed it from there using localhost.

First I set a breakpoint at line 71 of the hxp2019.js file.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/debug1.png" >}}

Stepping through the code line by line, we then get into the wasm code after line 73, i.e the wasm.check() function which passes the address where our input flag is stored and the length of the input. After this, on stepping into it, our code jumps into the wasm code.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/debug2.png" >}}

Stepping through each line (and after having done this over and over many times, I kind of understood what each line of the code was doing), we reach line 12 where actually our length of input is being checked with 50. So, we have to make our input length 50. We supply a dummy flag `hxp{45 times 'a'}`. Then we see that on stepping throght the code, and doing a lot of calculations on some array stored in memory, each character of our input is sequentially comapred with another character. The character to be compared with is loaded at line 284.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/debug3.png" >}}

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/debug4.png" >}}

Here, we see that the first character ('a' = 97) is to be compared with (109 = 'm'). What I did next, may not be the right way, but I was so excited that I had made progress was that I did this whole process 45 times, adding one character to my "flag" at a time until I had all characters of the flag. I had tried changing the code at line 288 to `br_if 1` but that seemed to crash somewhere. Anyways, whatever works during the CTF :stuck_out_tongue:.

The flag was - `hxp{merry_xmas___github.com/benediktwerner/rewasm}`

This could probably be the author of the chllenge as the repo is wasm reverse engineering tool. Loved the challenge!

----

# Junior CTF

## tracer - Forensics

File: [file](/post/2019-12-30_hxp-36c3-ctf/files/tracer)

The file looks like strace running on some process. I decided to scroll right to the very bottom and saw 

```
541   write(1, "\"Flag\"", 6)           = 6
541   write(1, " [New] 1L, 24C written", 22) = 22
541   write(3, "b0VIM 8.0\0\0\0\0\20\0\0\0\0\0\0\0\0\0\0\35\2\0\0root"..., 4096) = 4096
541   write(4, "# This viminfo file was generate"..., 1035) = 1035
```

This meant that at the end something was being written to a file named Flag using vim. I started looking at the preceeding lines and saw text or vim commands being typed in (i.e the read command). From line no. 65782, is the interetsing part. This has 'i' bein read, which is the command for insert in vim, that is typing began from here.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/vim.png" >}}

Now all I did was to focus on the `read` commands and type in whatever that was read on my local computer in vim. I treated `\33` as escape and just typed in whatever was being given as input as in the trace file.

Eventually I ended with some text which seemed meaningful, there was some slight error whic I fixed by intuition.

The flag was - `junior-nanoiswayBETTER!`

&nbsp;  

## maybe - Rev

File: [chal1](/post/2019-12-30_hxp-36c3-ctf/files/chal1)

We open up the file in Ghidra and head to the main function.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/rev11.png" >}}

Basically, if we see, the function is not doing anything, it is just taking our input of length 0x24 as a command line argument, then storing it at a +0x40 offset from a fixed string in memory, i.e. "junior-totally_the_flag_or_maybe_not". The rest of the computations don't mean anything as uvar3, ivar1, all are keeping the input unchanged. But the program still outputs "wrong!" and there does not seem to be any checking. 

After this I opened up GDB to analyse the flow. I set a breakpoint at the main function, and observed something interesting. 

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/re12.png" >}}

The fixed string "junior-totally_the_flag_or_maybe_not" is now changed to "ton_ebyam_ro_galf__flag_or_maybe_not". This has to be because of some code running before main. Heading back to Ghidra, I opened the `_INIT_0` and `_INIT_1` functions since they run before the entry point is reached. The `_INIT_1` function was the required code.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/re13.png" >}}

So, now after struggling for some time on the input evaluation part, I checked the `_FINI_0` and `_FINI_1` functions as well, as they run just before the end of the program. The `_FINI_1` function had the required code.

{{< figure src="/post/2019-12-30_hxp-36c3-ctf/images/re14.png" >}}

Here we see that the string "ton_ebyam_ro_galf__flag_or_maybe_not" is XORed with our input string at offset +0x40. This is then compared with alternate elements of the array `&DAT_003010a0`. The array contents are 

> b = [0x1E ,0x00 ,0x1A ,0x00 ,0x00 ,0x00 ,0x36 ,0x00 ,0x0A ,0x00 ,0x10 ,0x00 ,0x54 ,0x00 ,0x00 ,0x00 ,0x01 ,0x00 ,0x33 ,0x00 ,0x17 ,0x00 ,0x1C ,0x00 ,0x00 ,0x00 ,0x09 ,0x00 ,0x14 ,0x00 ,0x1E ,0x00 ,0x39 ,0x00 ,0x34 ,0x00 ,0x2A ,0x00 ,0x05 ,0x00 ,0x04 ,0x00 ,0x04 ,0x00 ,0x09 ,0x00 ,0x3D ,0x00 ,0x03 ,0x00 ,0x17 ,0x00 ,0x3C ,0x00 ,0x05 ,0x00 ,0x3E ,0x00 ,0x14 ,0x00 ,0x03 ,0x00 ,0x03 ,0x00 ,0x36 ,0x00 ,0x0F ,0x00 ,0x4E ,0x00 ,0x55 ,0x00]

So, all we have to do is XOR the fixed string with the alternate elements of this array and that should give us our flag.

```python
a = "ton_ebyam_ro_galf__flag_or_maybe_not"

b = [0x1E ,0x00 ,0x1A ,0x00 ,0x00 ,0x00 ,0x36 ,0x00 ,0x0A ,0x00 ,0x10 ,0x00 ,0x54 ,0x00 ,0x00 ,0x00 ,0x01 ,0x00 ,0x33 ,0x00 ,0x17 ,0x00 ,0x1C ,0x00 ,0x00 ,0x00 ,0x09 ,0x00 ,0x14 ,0x00 ,0x1E ,0x00 ,0x39 ,0x00 ,0x34 ,0x00 ,0x2A ,0x00 ,0x05 ,0x00 ,0x04 ,0x00 ,0x04 ,0x00 ,0x09 ,0x00 ,0x3D ,0x00 ,0x03 ,0x00 ,0x17 ,0x00 ,0x3C ,0x00 ,0x05 ,0x00 ,0x3E ,0x00 ,0x14 ,0x00 ,0x03 ,0x00 ,0x03 ,0x00 ,0x36 ,0x00 ,0x0F ,0x00 ,0x4E ,0x00 ,0x55 ,0x00]

flag = ''

b = b[::2]
for i in range(len(b)):
    flag += chr(b[i] ^ ord(a[i]))


print(flag)
# 'junior-alles_nur_kuchenblech_mafia!!'
```

The flag is - `junior-alles_nur_kuchenblech_mafia!!`

----

I had great fun solving this CTF. Learnt a ton! This was my last CTF and blog post for 2019.

2020 will see a lot more blog posts, writeups and some interesting security research too. Till then, sayonara :wave:.

<button id="openpopup">Subscribe to my posts!</button>

&nbsp;  

Follow me on [Twitter](https://twitter.com/shreyansh_26), [Github](https://github.com/shreyansh26) or connect on [LinkedIn](https://www.linkedin.com/in/shreyansh26/).