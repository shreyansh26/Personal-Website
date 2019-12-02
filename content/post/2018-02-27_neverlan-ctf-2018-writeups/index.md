---
title: "NeverLAN CTF 2018 Writeups"
authors: ["Shreyansh Singh"]
date: 2018-02-27T08:55:58.711Z
lastmod: 2019-11-29T21:04:19+05:30
tags: [ctf, information security, infosec, writeups]
categories: [Information Security]

summary: "These are the writeups of the problems I solved over the weekend for the NeverLAN CTF 2018."

featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "NeverLAN CTF 2018"
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

These are the writeups of the problems I solved over the weekend for the NeverLAN CTF 2018.


---

### **Scripting Challenges**
&nbsp;  
**1. Basic Math**

We are given a file with some numbers which we had to sum.


{{< figure src="/post/2018-02-27_neverlan-ctf-2018-writeups/images/2.png" caption="File" >}}

So, we write a simple python script to do it.

{{< gist shreyansh26 907d53ddd7a9b8b12c0e36ac4afef320 >}}


This gives the flag — 49562942146280612

&nbsp;  
**2. More Basic Math**

This time we have a larger list of numbers. However, we can just run the script again on the new file.

This gives us the flag — 50123971501856573397

&nbsp;  
**3. Even more Basic Math with some junk**

In this file, we see that we have spaces, commas and even English words in between the file. Using any text editor, we replace the commas with a space, and then write a script to replace all spaces with new lines.

{{< gist shreyansh26 9c4443d59df7469ad8652dcc6676d0b1 >}}


Then we run our first script again. We find two or three English words which give Value Error when the script is run. For them, we can manually remove them.

Finally, we get the flag — 34659711530484678082

&nbsp;  
**4. JSON Parsing 1**

On analysing the file, we find that each line is a JSON. We have to find the 5 AV engines which had the highest detection ratio (not detection count) in that file.

We write the following script to do that —
 
{{< gist shreyansh26 68288cff647b17b45752c6c4602d2fea >}}


The last five in the list are —

{{< figure src="/post/2018-02-27_neverlan-ctf-2018-writeups/images/3.png" caption="High Detection Ratio AV engines" >}}


So the flag is — `SymantecMobileInsight,CrowdStrike,SentinelOne,Invincea,Endgame`

---

### **Reversing Challenges**
&nbsp;  
**1. Commitment Issues**

The first thing which came to my mind is to run `strings` on the file. I did, and got the flag —**flag{don’t_string_me_along_man!}**

---

### **Interweb Challenges**
&nbsp;  
**1. ajax_not_soap**

On inspecting the script(ajax) of the webpage, we find that the form compares our username and password with one that is stored at the endpoint `/webhooks/get_username.php`. On going to that link we find the username as `MrClean`.

Also the password is also checked by the endpoint `/webhooks/get_pass.php?username=*username*` Replacing _username_ with `MrClean` we get the password (also the flag) as **flag{hj38dsjk324nkeasd9}**

&nbsp;  
**2. the_red_or_blue_pill**

The page says we can either take the red pill(endpoint `?red` ) or the blue pill(endpoint `?blue` ) but not both. We enter the endpoint as `?red&blue` to get the flag as **flag{breaking_the_matrix…I_like_it!}**

&nbsp;  
**3. ajax_not_borax**

This problem is very similar to ajax_not_soap with the difference here that when we go to the endpoint `/webhooks/get_username.php?username=`, we are presented with a hash (c5644ca91d1307779ed493c4dedfdcb7). We use an online MD5 decryptor to get the value as `tideade`. Then, when we go to the endpoint `/webhooks/get_pass.php?username=tideade`, we get a base64 encoded string, which on decryption gives the flag as **flag{sd90J0dnLKJ1ls9HJed}**

&nbsp;  
**4. Das_blog**

First, when we are presented with a login page, we find that a testing credential is available as a comment in the HTML. We login using those credentials. Then, we find that the cureent permission is `DEFAULT`. We need `admin` permissions to view the flag. On inspecting the cookies, we find that there is a cookie `permission` which has its value as user. We use the **EditThisCookie plugin** to change its value to `admin`. On refreshing, we get the flag as a blog post **flag{C00ki3s\_c4n_b33_ch4ng3d\_?}**

---

### Passwords Challenges
&nbsp;  
**1. Encoding != Hashing**

We are given a pcap capture. We open this in Wireshark and analyse the HTTP packets using the `http` filter. On reading the contents of the filtered packets, we find the flag.

{{< figure src="/post/2018-02-27_neverlan-ctf-2018-writeups/images/4.png" caption="Wireshark Packets analysis" >}}


The flag is **flag{help-me-obiwan}**

---

### **Trivia Challenges**
&nbsp;  
**1. Can you Name it?**

**Problem**— This system provides a reference-method for publicly known information-security vulnerabilities and exposures.

**Answer**— [Common Vulnerabilities and Exposures](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures)

&nbsp;  
**2. Can you find it? (Bonus)**

**Problem**— This Vulnerability was used for a major worldwide Ransomware attack. It was so bad it forced the software company to write a patch for end of life systems that they had stopped supporting years before the attack.

**Answer**— EternalBlue. And the ransomware was WannaCry.

&nbsp;  
**3. Yummy…**

**Problem**— These store small pieces of data sent from a website to the user’s computer. This yummy sounding things are stored by the user’s web browser while the user surfing the web. Answer is non-singular.

**Answer**— Cookies

&nbsp;  
**4. Can you find it?**

**Problem**— This Vulnerability was used for a major worldwide Ransomware attack. It was so bad it forced the software company to write a patch for end of life systems that they had stopped supporting years before the attack.

**Answer**— The formal listing code (CVE) for EternalBlue is **CVE-2017–0144**

&nbsp;  
**5. Can you search it?**

**Problem**— For the Vulnerability you found in question 2, There is a proof of concept. What is the string for TARGET_HAL_HEAP_ADDR_x64?

**Answer**— The vulnerability being discussed is EternalBlue. We canf ind the source code at [this link](https://gist.github.com/worawit/bd04bad3cd231474763b873df081c09a). There we find that TARGET_HAL_HEAP_ADDR_x64 is assigned **0xffffffffffd00010**

&nbsp;  
**6. Who knew?**

**Problem**— This product had Highest Number Of “Distinct” Vulnerabilities in 1999

**Answer**— A simple Google search of “Highest Number Of “Distinct” Vulnerabilities in 1999&#34;, gets us the following [link](https://www.cvedetails.com/top-50-products.php?year=1999). The product with the highest vulnerabilities was **Windows NT**

---

### **Blast from the Past Challenges**

**1. cookie_monster**

On inspecting the cookies, we find that the Cookie value should be the Red Guy’s name. We change the value of the cookie to `Elom`. On refreshing the page, we get the flag as **flag{C00kies_4r3_the_b3st}**
