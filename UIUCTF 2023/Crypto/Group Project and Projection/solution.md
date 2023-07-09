# Challenge description

```
Group Project

In any good project, you split the work into smaller tasks...

```
* Category: Cryptography
* Points: 50
* Author: Anakin

```
nc group.chal.uiuc.tf 1337
```

## Challenge File

<details>
    <summary>Chal.py (Click to Show/Hide)</summary>
 
```py
from Crypto.Util.number import getPrime, long_to_bytes
from random import randint
import hashlib
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad


with open("/flag", "rb") as f:
    flag = f.read().strip()

def main():
    print("[$] Did no one ever tell you to mind your own business??")

    g, p = 2, getPrime(1024)
    a = randint(2, p - 1)
    A = pow(g, a, p)
    print("[$] Public:")
    print(f"[$]     {g = }")
    print(f"[$]     {p = }")
    print(f"[$]     {A = }")

    try:
        k = int(input("[$] Choose k = "))
    except:
        print("[$] I said a number...")

    if k == 1 or k == p - 1 or k == (p - 1) // 2:
        print("[$] I'm not that dumb...")

    Ak = pow(A, k, p)
    b = randint(2, p - 1)
    B = pow(g, b, p)
    Bk = pow(B, k, p)
    S = pow(Bk, a, p)

    key = hashlib.md5(long_to_bytes(S)).digest()
    cipher = AES.new(key, AES.MODE_ECB)
    c = int.from_bytes(cipher.encrypt(pad(flag, 16)), "big")

    print("[$] Ciphertext using shared 'secret' ;)")
    print(f"[$]     {c = }")


if __name__ == "__main__":
    main()


```
</details>




## Understanding the challenge 


This challenge implements a simplified version of the Diffie-Hellman exchange protocol. 

First it generates the public parameters for the Diffie-Hellman key exchange: `g`, `p`, and `A`. `g` is set to `2`, while `p` is generated as a random `1024-bit` prime number using the getPrime function. A random integer `a` is then generated between `2` and `p-1`, and `A` is calculated as `g^a mod p`.

It then ask us to enter a value `k` which has some restrictions. If the value of `k` is `1`, `p-1`, or `(p-1)/2`, an error message is printed. 

Then it calculate the shared secret using the Diffie-Hellman key exchange algorithm. First, `Ak` is calculated as `A^k mod p`. Then, a random integer `b` is generated between `2` and `p-1`, and `B` is calculated as `g^b mod p`. Next, `Bk` is calculated as `B^k mod p`, and finally, the shared secret `S` is calculated as `Bk^a mod p`.

The shared secret is then used to derive an encryption key by taking its MD5 hash and using it as the key for an AES cipher in ECB mode.

The contents of the /flag file are encrypted using this cipher and converted to an integer value, which is stored in the variable c.

The ciphertext (the encrypted value of /flag) is then printed.


## Analysis

This is a CTF challenge and our task is to find the flag. From the code we know the following details.

* Value of `g`, `p`, and `A` 
* Value of `k` cannot be `1` ,`p-1`, or `(p-1)/2`

What we don't know:

* Value of `a` , `b` and `S` .

It is understood that without knowing the value of `S`, we actually decrypt the secret message.

So if we look at `S`, we can note that `Bk` , `a` and `p` are the values responsible for the `S` value. 

We know that `a` is random ,also we don't have control over any of the value expect `k` which indeed can be used to manipulate the value of `Bk` (i.e `Bk = B^k mod p`). 

So by inspection we can note that we know the value of `Bk` can be equal to `1` when `k = 0` and this value of `k` is allowed by the code, which results in know value of `S=1`

So this can be used now to decrypt the cipher text given by the code.

### Maths involved

When `k = 0`

$$ \mathrm Bk = B^0\ \bmod\ p\ =\ 1\ \bmod\ p\ =\ 1  $$

$$ \implies \mathrm S = 1^a\ \bmod\ p\ =\ 1\ \bmod\ p\ =\ 1  $$

**NOTE:**
There was an unintended solve where `k=p-1` also worked here which actually shouldn't be possible. 

Reson why `k = p-1` works mathematically is:

By Fermat’s Little Theorem if `p` is a prime number and `a` is an integer not divisible by `p`, then `a^(p-1) ≡ 1 (mod p)`. In other words, if you raise `a` to the power of `p-1` and then take the remainder when dividing by `p`, the result will always be `1`.

So by substituting `k = p-1`

$$ \mathrm Bk = B^{(p-1)}\ \bmod\ p\  $$

$$ \mathrm S = \big(B^{(p-1)}\big)^a\ \bmod\ p\ =\ 1\ \bmod\ p\ =\ 1 $$




## Solve Script

```py
from pwn import *
from Crypto.Util.number import long_to_bytes
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib

conn = remote('group.chal.uiuc.tf', 1337)

conn.recvuntil("[$] Did no one ever tell you to mind your own business??")

response = conn.recvuntil("Choose k =").decode()
g = int(response.split("g = ")[1].split("\n")[0])
p = int(response.split("p = ")[1].split("\n")[0])
A = int(response.split("A = ")[1].split("\n")[0])
k = p - 1  #can be 0 too. 
conn.sendline(str(k))
conn.recvuntil("\n")
response = conn.recvuntil("\n")
ciphertext=response.decode().split("c = ")[1].split("\n")[0]
c=int(ciphertext)
S = 1 
key = hashlib.md5(long_to_bytes(S)).digest()  
cipher = AES.new(key, AES.MODE_ECB)

flag = unpad(cipher.decrypt(long_to_bytes(c)), 16)  
print(flag)

conn.close()
```

## Output/Flag:

`uiuctf{brut3f0rc3_a1n't_s0_b4d_aft3r_all!!11!!}`


**This was part just part one, now comes the interesting version of it - Group Projection**

# Challenge description

```
Group Projection

I gave you an easier project last time. 
This one is sure to break your grade!

```
* Category: Cryptography
* Points: 50
* Author: Anakin

```
nc group-projection.chal.uiuc.tf 1337
```



**Solved by ctfguy from Team - CyberSpace**








