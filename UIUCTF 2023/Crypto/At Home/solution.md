# Challenge description

```
At Home

Mom said we had food at home

```
* Category: crypto
* Points: 50
* Author: Anakin

<br>

## Challenge File

<br>

<details>
    <summary>Chal.py (Click to Show/Hide)</summary>

```py
from Crypto.Util.number import getRandomNBitInteger

flag = int.from_bytes(b"uiuctf{******************}", "big")

a = getRandomNBitInteger(256)
b = getRandomNBitInteger(256)
a_ = getRandomNBitInteger(256)
b_ = getRandomNBitInteger(256)

M = a * b - 1
e = a_ * M + a
d = b_ * M + b

n = (e * d - 1) // M

c = (flag * e) % n

print(f"{e = }")
print(f"{n = }")
print(f"{c = }")
```
</details>

<br>

<details>
    <summary>Chal.txt (Click to Show/Hide)</summary>
    
```
e = 359050389152821553416139581503505347057925208560451864426634100333116560422313639260283981496824920089789497818520105189684311823250795520058111763310428202654439351922361722731557743640799254622423104811120692862884666323623693713
n = 26866112476805004406608209986673337296216833710860089901238432952384811714684404001885354052039112340209557226256650661186843726925958125334974412111471244462419577294051744141817411512295364953687829707132828973068538495834511391553765427956458757286710053986810998890293154443240352924460801124219510584689
c = 67743374462448582107440168513687520434594529331821740737396116407928111043815084665002104196754020530469360539253323738935708414363005373458782041955450278954348306401542374309788938720659206881893349940765268153223129964864641817170395527170138553388816095842842667443210645457879043383345869

```

</details>

<br>

## Overview 
<br>

The `chal.py` generates an RSA public key `(e, n)` and a ciphertext `c` by encrypting a flag. We are given with the `e,n,c` values in `chal.txt` and our task is to find the flag.

## Analysis

First it converts the flag, which is a string of bytes, into an integer. The `int.from_bytes` method takes two arguments: the first is the byte string to convert, and the second is the byte order to use. In this case, the byte order is `"big"`, which means that the most significant byte is at the beginning of the byte string. (**NOTE**: This is just a sample flag)

Then it generates four random `256-bit` integers `a , b , a_ , b_ ` using the `getRandomNBitInteger` function.

Then some simple mathematical operations are performed on these value to get `M ,e ,d , n` 

We now encrypt the flag using the RSA public key `(e, n)` to produce the ciphertext `c`. The encryption is done by raising the flag to the power of e modulo n :

$$ \mathrm {c = (flag\ *\ e)\ \bmod n} $$

Also `e` is a `231` digit positive integer, `n` is a `308` digit positive integer and `c` is a `293` digit positive integer.


## Math is fun

After analysing the problem and understanding it, all i saw is bunch of math equations and numbers all around. 

So first i approached this problem mathematically.

Let’s say that the original message `(flag)` is represented by the integer `m`. The encryption process can be represented mathematically as $\mathrm{c ≡ m^e (\bmod\ n)}$ .

We can note that `e` is coprime with `n`.
<details>
    <summary> <b> How? (Try or Click here to know) </b> </summary>

<br>

In general, it is not guaranteed that any two randomly chosen integers will be coprime. However, in this specific case, we can prove that `e` and `n` are coprime based on how they are generated in chal.py.

Let’s take a closer look at how `e` and `n` are generated in `chal.py`. We have:

$$\mathrm{ M = a * b - 1}$$
$$\mathrm{ e = a\_ * M + a}$$
$$\mathrm{ d = b\_ * M + b}$$
$$\mathrm{ n = (e * d - 1)\ //\ M}$$

 From the above equation we can write

$$\mathrm{ e*d - 1 = k*M}$$

for some intger `k`.

$$\implies \mathrm{ e*d -1 = k(a*b-1)}$$

$$\implies \mathrm{ e*d -1 = k*a*b-k}$$
  
Now, let’s consider the greatest common divisor of `e` and `n`. 

Since `n` is a factor of $\mathrm{e.d - 1}$, we have 

$$\mathrm{gcd(e, n)\ |\ (e * d - 1)}$$

 From the equation above, we also have 

 $$\mathrm{(e * d - 1)\ |\ (k * a * b - k)}$$
 
Combining these two facts, we get:

$$\mathrm{gcd(e, n)\ |\ (k * a * b - k)}$$

Now, let’s assume for the sake of contradiction that $\mathrm{gcd(e, n) > 1}$ . This means that there exists a prime number `p` such that $\mathrm{p\ |\ gcd(e, n)}$ . 

Since $\mathrm{gcd(e, n)\ |\ (k * a * b - k)}$ , we also have $\mathrm{p\ |\ (k * a * b - k)}$ . This means that either $\mathrm{p\ |\ k}$ or $\mathrm{p\ |\ (a * b - 1)}$ .

If $\mathrm{p\ |\ k}$, then we can write $\mathrm{k = p * q}$ for some integer `q`. 

Substituting this into the equation above, we get:

 $$\mathrm{(e * d - 1) = p * q * a * b - p * q}$$
 
Since $\mathrm{p\ |\ (e * d - 1)}$ and $\mathrm{p\ |\ (p * q)}$ , it follows that $\mathrm{p\ |\ (q * a * b)}$ . 


This means that either $\mathrm{p\ |\ q}$ , or $\mathrm{p\ |\ a}$ , or $\mathrm{p\ |\ b}$ .

If $\mathrm{p\ |\ q}$ , then we can write $\mathrm{q = p * r}$ for some integer `r`. Substituting this into the equation above, we get:
 $$\mathrm{(e * d - 1) = p^2 * r * a * b - p^2 * r}$$
 
  Since both terms on the right-hand side are divisible by $\mathrm{p^2}$, it follows that $\mathrm{(e*d-1)}$ is divisible by $\mathrm{p^2}$ .
  
   However, this contradicts our assumption that `p` is a prime divisor of $\mathrm{gcd(e,n)}$ , since if $\mathrm{gcd(e,n)}$ is divisible by $\mathrm{p^2}$ then it must be divisible by `p` as well.

If either $\mathrm{p\ |\ a}$ or $\mathrm{p\ |\ b}$ then since $\mathrm{e=a_*M+a}$ and $\mathrm{M=a*b-1}$ it follows that `e` is divisible by `p`. However this contradicts our assumption that $\mathrm{gcd(e,n)>1}$ since if `e` is divisible by `p` then $\mathrm{gcd(e,n)}$ must be divisible by `p` as well.

Therefore, our assumption that $\mathrm{gcd(e,n)>1}$ must be false. This means that $\mathrm{gcd(e,n)=1}$ and hence `e` and `n` are coprime.


</details>

<br>

Since `e` and `n` are coprime, there exists an integer `d` such that

$$\mathrm{e*d ≡ 1 (\bmod\ n)}$$

This means that we can write $\mathrm{e*d = k.n + 1}$ for some integer `k`. Multiplying both sides of this equation by `m`, we get:

 $$ \mathrm{(m^e)^d ≡ m  (k*n + 1)} $$

Now, let’s consider the LHS of this equation modulo `n`. 
We have: 
$$\mathrm{(m^e)^d ≡ c^d\ (\bmod\ n)}$$ 
Because, $\mathrm{c ≡ m^e\ (\bmod\ n)}$ .

In the RHS we have: 
$$\mathrm{m (k*n + 1) ≡ m\ (\bmod\ n)}$$ 

Because $\mathrm{(k*n) ≡ 0\ (\bmod\ n)}$ .

Putting these two results together, we get: 

$$\mathrm{(m^e)^d\ ≡\ c^d\ ≡\ m\ (\bmod\ n)}$$

$$\boxed{\mathrm{m ≡\ c^d\ \bmod\ n }}$$

So, Let proceed to code based on this.

## Solve Script
<br>

```py
from Crypto.Util.number import long_to_bytes


e = 359050389152821553416139581503505347057925208560451864426634100333116560422313639260283981496824920089789497818520105189684311823250795520058111763310428202654439351922361722731557743640799254622423104811120692862884666323623693713
n = 26866112476805004406608209986673337296216833710860089901238432952384811714684404001885354052039112340209557226256650661186843726925958125334974412111471244462419577294051744141817411512295364953687829707132828973068538495834511391553765427956458757286710053986810998890293154443240352924460801124219510584689
c = 67743374462448582107440168513687520434594529331821740737396116407928111043815084665002104196754020530469360539253323738935708414363005373458782041955450278954348306401542374309788938720659206881893349940765268153223129964864641817170395527170138553388816095842842667443210645457879043383345869


d = pow(e, -1, n)
m = (c * d) % n
flag = long_to_bytes(m)
print(flag)
```

## Ouput/Flag

`uiuctf{W3_hav3_R5A_@_h0m3}`



 

