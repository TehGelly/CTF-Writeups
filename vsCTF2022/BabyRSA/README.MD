# Baby RSA

### Description
Just a baby challenge...

### Exploration
```py
p = getPrime(128)
q = getPrime(128)

while p % e != 1:
    p = getPrime(128)
while q % e != 1:
    q = getPrime(128)

n = p * q
m = bytes_to_long(flag.encode())
c = pow(m, e, n)
print(f"Ciphertext: {hex(c)}")

# Ciphertext: 0x459cc234f24a2fb115ff10e272130048d996f5b562964ee6138442a4429af847
```

According to the challenge, this _looks_ terribly similar to textbook RSA - generate a semiprime modulus **n** with two large primes, use a public exponent **e**, and one would decrypt with the private exponent **d**, calculated pretty easily once you know the factorization of **n**. 

Using RsaCtfTool (https://github.com/RsaCtfTool/RsaCtfTool), I dump the **n** and **e** from the file (and don't get the factorization from the tool itself, unfortunately).

```py
n = 52419317100235286358057114349639882093779997394202082664044401328860087685103
e = 101
```

Other than being a slightly nonstandard exponent, **e** looks fine, and given **n** is relatively short, it should be simple enough to factorize **n**, calculate the private exponent **d** from **pow(e,-1,phi(n))**, and finally, getting the **pt** with **pow(ct,d,n)**. My installations of yafu are out of commission at the moment, so the best I could do for a mindless factorization was using Alpertron's online ECM (https://www.alpertron.com.ar/ECM.HTM). After letting the browser chug a bit, I finally got the factors I needed!

```py
p = 184980129074643957218827272858529362113
q = n//p # = 283378097758180413812138939650885549231
phiN = (p-1)*(q-1)
d = pow(e, -1, phiN) # ValueError: base is not invertible for the given modulus
```

Oh beans. I misread the checks for **e**, as usually you want to _avoid_ **phi(n)** being divisible by **e** - otherwise, you no longer have unique decryption. Those checks would normally look like:

```py
while p % e == 1:
    p = getPrime(128)
while q % e == 1:
    q = getPrime(128)
```

There's quite a few references online about non-unique decryption - the Rabin Cryptosystem, which is _very_ well laid out as long as **e**=2, or, if **e** is small enough, retrieving 1/**e** of the bits. But, since **e** is 101, having 1/101th of the plaintext is probably more than useless.

### Solution
After a long time researching different mathoverflow posts and such, I finally stumbled across "Incorrectly Generated RSA Keys" by Daniel Shumow (https://eprint.iacr.org/2020/1059.pdf) that laid out this process, assuming only _one_ of p-1, q-1 was divisible by **e**:
* Create the decryption exponent **d** with `d = pow(e, -1, phi(n)//e)`, and calculate `a = pow(ct,d,n)`. **a** is a valid plaintext, given the criteria.
* Create a generator **gE** with some small integer **m** via `gE = pow(m,phi(n)//e,N)`, and confirm **gE** != 1 mod **n**
* Multiply **a** by **gE** enough times until you find a plaintext to your liking. This process will loop through all **e** plaintexts.

There's a couple problems with this process when applied to this problem. First, we have to reduce **phi(n)** by **e**^2, which is simple enough, though likely busts the underlying math. Fingers crossed.
Second, we don't just have 101 plaintexts that are possible - since both p-1 _and_ q-1 are divisible by **e**, we have 101*101 = 10201 plaintexts to search through! If a generator only goes through **e** of them, how do I find the rest?

After some testing, it turns out that for different values of **m**, you get different generators that go through different groups of size **e**. While there's no guarentees that **gE(m)** and **gE(m+1)** won't go through the same group, it's likely possible to just go through a bunch of **m** and find each different plaintext until there's one that follows the flag format. Since every flag in this CTF starts with "vsCTF{", we have our filter for plaintexts, and thus our method:

### Solution script

```py
#provided to us from pubkey.pem and chall.py
n = 52419317100235286358057114349639882093779997394202082664044401328860087685103
e = 101
ct = 0x459cc234f24a2fb115ff10e272130048d996f5b562964ee6138442a4429af847

#alpertron ecm for the factorization
p = 184980129074643957218827272858529362113
q = n // p
phiN = (p-1)*(q-1)

#we have to reduce phiN by e until we get a valid inverse
reducedPhiN = phiN//(e*e)
reducedD = pow(e, -1, reducedPhiN)

#base plaintext a
a = pow(ct, reducedD, n)
pt = b''
for m in range(2,250):
    #create random generators, as none make it to all 10201 solutions
    ge = pow(m,reducedPhiN, n)
    if ge == 1:
        break
    #we'll use a tmp variable to preserve a
    l = ge
    #loops through the E group for this generator gE
    for j in range(101):
        pt = ((a*l)%n).to_bytes(32,'big')
        #check if it's the plaintext we're looking for
        if b'vsctf' in pt:
            break
        l *= ge
        l %= n
    if b'vsctf' in pt:
        print(pt)
        break 
        
#b'\x00\x00\x00vsctf{5m411_Pr1m3_15_Un54f3!}'
#m = 53 (could have been worse)
```

Note: This is definitely not the method I would say is the _correct_ one - not by a long shot. Many people solved this with less esoteric methods. Sagemath seemingly has magicks that allow for 3 liners that automagically provide all plaintexts, without explicitly calculating the factorization of n. This solution here seems to generalize well enough with even more factors of **e** in **phi(n)**, but I have neither the confidence nor the interest to prove as much.

Flag: **vsctf{5m411_Pr1m3_15_Un54f3!}**
