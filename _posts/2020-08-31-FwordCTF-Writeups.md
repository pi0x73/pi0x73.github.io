---
layout: single
title: "Fword CTF 2020 Writeups"
excerpt: "Writeups for 2 of the challenges from Fword 2020 CTF"
date: 2020-08-31
header:
  teaser: /assets/images/fword-ctf/ctf.png
  teaser_home_page: true
categories:
  - ctf
tags:
  - crypto
  - osint
---

## Overview
This was quite a good thought CTF with many interesting and tough challenges which I enjoyed while playing , however going to only make 2 challenge writeups on the following blog post.
These ones looked very interesting to me.

### One Part! [CRYPTOGRAPHIC]
****Author : Semah BA****

**Description** :
>*One part is secure ?*

We are only given the remote server to connect on , and see what is going on with that one challenge : ``onepart.fword.wtf 4445``.

We can use ****netcat**** to connect to the remote server and grab some information :
```sh
pwn@ubuntu:~/CTF/fword$ nc onepart.fword.wtf 4445

	Welcome to my secure Land !!! 
	Everything is in front of you !
	test it for me before publishing it 
		
you public pair : (15613468538689026439789035486926660047933091470223229842635697103266750671565247899488275163605402661984712863965372738639126665753096382260138447736228761725912673459635216242677879243135336946097666618761528068923793831510160310385158724757828758463719323195622964651321321436661271206326831849706532562749448093035253275086206243790986960018831679175778998525917929677081854589592676599479540041321636029038922291243937491659445003899125612203361279993735563033218034372866222751814817465849046177301107417914106768656120464448050600512949716699761244228618417154785251943994879914402862490734801951488945740459361, 65537)
Bonus information 
dp : 104402896302994202734948778517366637140776236526426804228566136031503470616299834533776886961654191814306823291837544844754349730075760901149959051770868411686530924029853963683561365410378706351590616730419408741611270189590310647046764511499366439992610243326864810499569332114189408231246271434449891104903
Cipher : 9045242622763274357544835822239980048874156443880644385138675500759584604526273475226842750416108701427998940567271877957754330185397605308429857013804586694230860812067586635816236827789043509539544108153164004825156286367834342137620394360377782112359204843336537521290071551705674877988593444673896840081020594848648892508563525548629538386215847502296041252724965402173421681679607664531325094223651810864485017095591981097893070155871256153570767852661054256846500272829720800455981893631803581317717280593572533847348974018714387685164187112725078714544306615308363939229011022605751769677183577421294695836989
```

From the first review We can see that we are given enough values to perform a ``partial key exposure attack`` therefore we can use ****Chinese Remainder Theorem (CRT)**** for the decryption, which is quite similar to the usual RSA decryption but with one different condition.

The difference from the usual decryption and the RSA-CRT decryption is that here We use :

****dp≡d(modp−1)****

instead of a single private key (d).

From some googling about the CRT-RSA decryption process I found the following information :

>*If used correctly, this allows for speedup when compared to normal RSA decryption, since two smaller modular exponentiations are performed with significantly smaller exponents.
So as long as we know the lower half of the Least Significant Bits (LSBs) of dp, and e is of size poly(log(N)), we can get the factorization of the modulus in polynomial time.*

We are already given the ***dp*** and ***e*** value, so that would make it easier for us to break the cipher quickly. 
Having leaked the ****dp**** value, it means that I am already given the whole part of the private key so I have enough values to do the decryption.

Moreover I found this really interesting and useful explaination over the CRT decryption process : [PDF](
https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjm0OCLo8brAhXMiFwKHZqiD0QQFjAXegQIChAB&url=https%3A%2F%2Fdigitalcommons.aaru.edu.jo%2Fcgi%2Fviewcontent.cgi%3Farticle%3D1081%26context%3Dfcij&usg=AOvVaw3MdyU9f6exNWAP0ysWiKhg).

From the process we know that : ``edp=1(modp−1)``

To rrearange to our needs that would be : ``edp−1=0(modp−1)``  meaning that ``edp − 1`` divides ``p−1``.

I can rewrite this as :
``edp−1=k(p−1)``, where ***k∈N*** and ***k<e***, since k(p−1) is a multiplication of edp − 1.

For ****p**** we get : ``p = edp−1+k / k``

Since the value of `e=65537`, it is possible to try every ****k**** in range of ****e**** until we get a value of ****k**** so that `Np=⌊Np⌋`, so we can make sure we have obtained the prime factor ****q**** and thus the factorization of ****N****.

To uncipher the given cipher I made the following ***Python3*** script based on the steps and equations explained above :

```python
import binascii
import string


N = 211767290324398254371868231687152625271180645754760945403088952020394972457469805823582174761387551992017650132806887143281743839388543576204324782920306260516024555364515883886110655807724459040458316068890447499547881914042520229001396317762404169572753359966034696955079260396682467936073461651616640916909
dp = 10169576291048744120030378521334130877372575619400406464442859732399651284965479823750811638854185900836535026290910663113961810650660236370395359445734425
e = 65537
c = 42601238135749247354464266278794915846396919141313024662374579479712190675096500801203662531952565488623964806890491567595603873371264777262418933107257283084704170577649264745811855833366655322107229755242767948773320530979935167331115009578064779877494691384747024161661024803331738931358534779829183671004

def random(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = random(b % a, a)
        return (g, x - (b // a) * y, y)

def modinv(a, m):
    g, x, y = random(a, m)
    if g != 1:
        raise 'Does not exist'
    else:
        return x % m

def factorize():
    for k in range(2,e):
        p = (e * dp - 1 + k) // k
        if N % p == 0:
            return p
    return -1

p = factorize()
q = N // p

phi = (p - 1) * (q - 1)
d = modinv(e,phi)
print(binascii.unhexlify(hex(pow(c,d,N))[2:])
```
After executing the script in my machine I get the following results : 

![img](https://raw.githubusercontent.com/pi0x73/pi0x73.github.io/master/assets/images/fword-ctf/result.png)

>*Flag : flag{wow_leaking_dp_breaks_rsa?_32697643574}*



























