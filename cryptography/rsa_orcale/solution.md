## Problem Description

link: https://play.picoctf.org/practice/challenge/422

Can you abuse the oracle? An attacker was able to intercept communications between a bank and a fintech company. They managed to get the message (ciphertext) and the password that was used to encrypt the message.

Additional details will be available after launching your challenge instance.

## Background of RSA

Find two very very large primes $p, q$. Let

$n = p * q$ 

$m = (p-1) * (q-1)$.

Choose $0 < d < m$ that is coprime to m, i.e. $(d, m) = 1$, there exists $0 < e < m$ such that 

$de \equiv 1 \,\, (mod\,\, m)$

Then the public key is $(n, d)$ and the private key is $(n, e)$. If we want to send message $a$, we encrypt it as

$b \equiv a^d \,\, (mod\,\, m)$.

The receiver can decrypt it with the private key

$a = b^e \,\, (mod\,\, m)$.

The reason behind is Fermat's little theorem, which says for arbitrary prime number $r$, if $c$ not divisible by $r$, then $c^{(r-1)} \equiv 1 \,\, (mod\,\, r)$.

This encryption cannot be decoded with the private key, because $n$ is so big, it is computationally impossible to find $p$, $q$, and thus m 
so given $d$, one has no idea how to find $e$. 

## Solution

The idea follows from this youtube video https://www.youtube.com/watch?v=XsiwqgGourA explained by Prof. Martin Carlisle 

We have a password $b$ (encrypted from an unknown $a$) that is encoded. The oracle on the server allows us to encrypt and decrypt almost anything, however, the only restriction is that we cannot decode $b$ directly to get $a$. 

So the idea is, we first encrypt \x02, the oracle should give us $g$ where

$2^e \equiv g\  (\text{mod } n)$

Then if we try to decrypt $g * b$, we will get 

$(g * b)^d \equiv g^d * b^d \equiv 2a \, \, (\text{mod } n)$ (note that this works when $2a < n$, which is generally true since $n$ is so big)

Finally, we divide the result $2a$ by $2$ to get the decrypted password.  

## Code

```
from pwn import *

# encrypt \x02

conn = remote("titan.picoctf.net", PORT)
print(conn.recv().decode())
conn.sendline(b'E')
print(conn.recv().decode())
conn.sendline(b'\x02')
conn.recvuntil('mod n)')
encrypted_string = conn.recvline()
encrypted_number = int(encrypted_string.decode())

# read the password

with open("password.enc") as file:
    password = file.read()
password = int(password)

# decrypt the password by multiplying with the encrypted \x02

conn.sendline(b'D')
print(conn.recv().decode())
print(conn.recv().decode())
to_decrypt = encrypted_number * password
conn.sendline(str(to_decrypt).encode())
# conn.recv
conn.recvuntil('mod n): ')
decipher = conn.recvline().decode()
decipher = int(decipher, 16) // 2
print(bytes.fromhex(hex(decipher)[2:]).decode('ascii'))
```

Once we obtain the decrypted password: KEY (the answer is not included, as you should try it yourself), we can run the following on Linux to get the answer. (For Windows, you can use tools like Git Bash.)

`openssl enc -aes-256-cbc -d -in secret.enc -k KEY`

