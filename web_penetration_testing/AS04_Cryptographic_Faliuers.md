# Overview
Machine : [Application Design Flaws](https://tryhackme.com/)
Platform : [Tryhackme](https://tryhackme.com/)
Difficulty : easy
Category : Web

this machine focus on encryption when use it wrongly 

---
# Scope 
Target IP : 10.49.145.228
In scope services : 

``` 
http://10.49.145.228:5004/ 
```
---
# Enumeration
After we deployed the machine 10.49.145.228
let's investigate [[#^31209b|machine]]

^31209b

![[Pasted image 20260215111824.png]]
#### Misconfiguration
encrypted Document : 
```
Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe
```
^b1

First i intercept **HTTP REQUEST & RESPONSE** for this [[#^31209b|machine IP]] to view source code if there client-side source code we can use

![[Pasted image 20260215135329.png]]

there is GET method with interesting file name let's open it

![[Pasted image 20260215112144.png]]
there is unhidden important variables . Breaking those variables down we can notice that 
```
const ENCRYPTION_MODE = "ECB";
```
ECB is week [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) Cipher mode encryption
Lets decrypt it using [[#^b1|misconfiguration]] i will use this online [tool](https://anycript.com/crypto) . we will just fill those boxes using [[Pasted image 20260215135329.png|This client-side js file]]
![[Pasted image 20260215142513.png]]
we got the flag!

# Lessons Learned

don't use encryption incorrectly 

# Recommendations 

Use strong modern algorithms such as AES-GCM, ChaCha20-Poly1305, or enforce TLS 1.3 with valid certificates