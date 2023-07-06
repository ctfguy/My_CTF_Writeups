# Challenge description

```
Morphing Time

The all revealing Oracle may be revealing a little too much...
```
* Category: Cryptography
* Points: 50
* Author: Anakin

```
nc morphing.chal.uiuc.tf 1337
```
<details>
    <summary>Chal.py (Click to Show/Hide)</summary>

```python
#!/usr/bin/env python3
from Crypto.Util.number import getPrime
from random import randint

with open("/flag", "rb") as f:
    flag = int.from_bytes(f.read().strip(), "big")


def setup():
    # Get group prime + generator
    p = getPrime(512)
    g = 2

    return g, p


def key(g, p):
    # generate key info
    a = randint(2, p - 1)
    A = pow(g, a, p)

    return a, A


def encrypt_setup(p, g, A):
    def encrypt(m):
        k = randint(2, p - 1)
        c1 = pow(g, k, p)
        c2 = pow(A, k, p)
        c2 = (m * c2) % p

        return c1, c2

    return encrypt


def decrypt_setup(a, p):
    def decrypt(c1, c2):
        m = pow(c1, a, p)
        m = pow(m, -1, p)
        m = (c2 * m) % p

        return m

    return decrypt


def main():
    print("[$] Welcome to Morphing Time")

    g, p = 2, getPrime(512)
    a = randint(2, p - 1)
    A = pow(g, a, p)
    decrypt = decrypt_setup(a, p)
    encrypt = encrypt_setup(p, g, A)
    print("[$] Public:")
    print(f"[$]     {g = }")
    print(f"[$]     {p = }")
    print(f"[$]     {A = }")

    c1, c2 = encrypt(flag)
    print("[$] Eavesdropped Message:")
    print(f"[$]     {c1 = }")
    print(f"[$]     {c2 = }")

    print("[$] Give A Ciphertext (c1_, c2_) to the Oracle:")
    try:
        c1_ = input("[$]     c1_ = ")
        c1_ = int(c1_)
        assert 1 < c1_ < p - 1

        c2_ = input("[$]     c2_ = ")
        c2_ = int(c2_)
        assert 1 < c2_ < p - 1
    except:
        print("!! You've Lost Your Chance !!")
        exit(1)

    print("[$] Decryption of You-Know-What:")
    m = decrypt((c1 * c1_) % p, (c2 * c2_) % p)
    print(f"[$]     {m = }")

    # !! NOTE !!
    # Convert your final result to plaintext using
    # long_to_bytes

    exit(0)


if __name__ == "__main__":
    main()

```

</details>

<br>

## Overview

<br>

The `chal.py` given here is an implementation of the ElGamal encryption system, which is an asymmetric key encryption algorithm for public-key cryptography based on the Diffie–Hellman key exchange. 

## Understanding the challenge

The first step in solving a challenge is understanding the problem which is really important. So lets see whats happening in the above code.

```py
    g, p = 2, getPrime(512)
    a = randint(2, p - 1)
    A = pow(g, a, p)
```

Firt the [generator](https://crypto.stackexchange.com/questions/16196/what-is-a-generator) `g` and the prime `p` are generated in the `main` function. Then it generates the private key `a` and the public key `A` . (These two are the direct implementation of the `setup` and `key` function without actually calling those two function in `main`). 

```py
decrypt = decrypt_setup(a, p)
encrypt = encrypt_setup(p, g, A)
```

Here a `decrypt` function is created using the `decrypt_setup` function, which takes in the private key `a` and the prime `p` and The `encrypt` function is created using the `encrypt_setup` function, which takes in the prime `p`, the generator `g`, and the public key `A`. 

Then it prints the values of `p` ,`g` and `A`.

```py
    c1, c2 = encrypt(flag)
    print("[$] Eavesdropped Message:")
    print(f"[$]     {c1 = }")
    print(f"[$]     {c2 = }")
```

This is part which encrypts the flag into two ciphertexts and gives us the ciphertexts.

```py
    try:
        c1_ = input("[$]     c1_ = ")
        c1_ = int(c1_)
        assert 1 < c1_ < p - 1

        c2_ = input("[$]     c2_ = ")
        c2_ = int(c2_)
        assert 1 < c2_ < p - 1
    except:
        print("!! You've Lost Your Chance !!")
        exit(1)

```

The program the asks us two ciphertexts `c1_` and `c2_` as input and check if both of them are in the range `(1,p-1)` . If they are within the range it proceeds or print the error message and exits.


```py

    m = decrypt((c1 * c1_) % p, (c2 * c2_) % p)
    print(f"[$]     {m = }")
```

We get to this part of the code if we give inputs `c1_` and `c2_` which satifies the range check. 

`m` calls the `decrypt` function of `decrypt_setup` with two argument `(c1 * c1_) % p` and `(c2 * c2_) % p`.

So basically this line is modifying the original ciphertext by multiplying its parts with the values `c1_` and `c2_` we enter and then passing the modified ciphertext to the decrypt function to obtain the decrypted message `m`.

<br>

## Analysis

The ElGamal encryption system is an asymmetric key encryption algorithm for public-key cryptography based on the Diffie–Hellman key exchange . Here the flag $\mathrm{f}$ is encrypted using a public key $\mathrm{A}$ to produce ciphertext $\mathrm{(c_{1}, c_2)}$ where $\mathrm{c_1 = g^k \bmod\ p}$ and $\mathrm{c_2 = f*A^K \bmod\ p}$ .

The ElGamal encryption system is homomorphic under multiplication, which means that the product of two ciphertexts encrypting two messages $\mathrm{m_1}$​ and $\mathrm{m_2}$​ is a valid ciphertext that encrypts the product of the two messages $\mathrm{m_1​∗m_2}$​ . This can be seen from the way the second part of the ciphertext is computed during encryption: $\mathrm{c_2​=f∗A^k\bmod\ p}$, where $\mathrm{A^k}$ is the shared secret value computed as $\mathrm{A^k=h^y\bmod\ p}$. If we multiply two such values together, we get:
$$\mathrm{(m_1​.A^k)(m_2​.A^k)\bmod\ p=(m1​.m2​)(A^k)^2\bmod\ p}$$ 

which is a valid ciphertext for the message $\mathrm{m1​∗m2}$ ​.

In this challenge, we are given the opportunity to interact with an oracle that takes in two values `c1_`  and `c2_`  and returns the decrypted value of the modified ciphertext `(c1 * c1_)` .So it seems like the goal is to use this oracle to obtain the flag.

One way to approach this problem is to provide values for `c1_` and `c2_` that effectively cancel out the effect of the public key on the ciphertext. Since the second part of the ciphertext is computed as $\mathrm{c_2​=m∗A^k\bmod\ p}$ , ot is easy to noice that providing a value for `c2_` that is equal to the inverse of $\mathrm{A^k\bmod\ p}$ will cancel out its effect on the ciphertext, leaving only the original message value. 

To compute the inverse of $\mathrm{A^k \bmod\ p}$ , we can use the fact that $\mathrm{A=g^a\bmod\ p}$ , where $\mathrm{a}$ is the private key. This means that $\mathrm{A^k=g^{ak}\bmod\ p}$ . Since we know that $\mathrm{c_1​=g^k\bmod\ p}$ , we can compute the inverse of $\mathrm{A^k\bmod\ p}$ as $\mathrm{(c_1a​)^{−1}\bmod\ p}$ . However, since we do not know the value of the private key $\mathrm{a}$ , we cannot compute this value directly.

Instead, we can provide a value for `c1_` that is equal to $\mathrm{g}$ , which effectively multiplies $\mathrm{c_1}$ by $\mathrm{g}$ . This means that the first part of the modified ciphertext passed to the decrypt function will be $\mathrm{(c_1∗g)}$ . Since the decrypt function computes s as $\mathrm{(c_1∗g)a}$ , this means that providing a value for `c2_` that is equal to $\mathrm{A}$ will effectively multiply $\mathrm{c_2}$ by $\mathrm{A}$ , resulting in a second part of the modified ciphertext equal to $\mathrm{(c_2∗A)}$ . Since these two parts of the modified ciphertext are passed to the decrypt function, which computes $\mathrm{m}$ as $\mathrm{(c_2∗A)∗(g^{a(k+1)})^{−1}}$ , we obtain the original message value.

In summary, by providing values for `c1_` and `c2_` that are equal to $\mathrm{g}$ and $\mathrm{A}$ , respectively, we can effectively cancel out the effect of the public key on the ciphertext and obtain the original message value using only mathematical operations. 

Now lets prcoeed to solve this based on our analysis.

<br>

## Solve Script

```py
from Crypto.Util.number import long_to_bytes
from pwn import *
conn = remote('morphing.chal.uiuc.tf', 1337)
conn.recvuntil("[$] Welcome to Morphing Time")
response = conn.recvuntil("c1_ =").decode()

A = int(response.split("A = ")[1].split("\n")[0])
conn.sendline(str(2))                                  #Because g is always set to 2.

conn.recvuntil("c2_ =").decode()
conn.sendline(str(A))
conn.recvuntil("[$] Decryption of You-Know-What:")
conn.recvline()
value=str(conn.recvline())

m = int(value.split("m = ")[1].split("\\")[0])

print(long_to_bytes(m))
conn.close()

```

The script does nothing complex. It just send `c1_ = g = 2`
and `c2_ = A` (The reason why it works is mentioned in the analysis) after getting their values from the response. 

Then it reads the value of `m` that we get as response after sending the value and converts it to bytes as per the note given in the challenge code to get the flag.  

This is simple and can also be done manually by giving the correct inputs to the connection and getting `m` value which when converting it to bytes gives us the flag . 


## Output/Flag

`uiuctf{h0m0m0rpi5sms_ar3_v3ry_fun!!11!!11!!}`

<br>


**Solved by ConnorM from Team - CyberSpace**

