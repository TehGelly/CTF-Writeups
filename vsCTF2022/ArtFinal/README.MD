# Baby RSA

### Description
School is wrapping up, and I wanted to add some pizzazz to my Art Final, I hope you like it!

### Exploration
```py
for i in range(boring.size[0] * boring.size[1]):
    rgba = tuple(random.randbytes(4))

key = bytes(random.sample(random.randbytes(16), 16))
iv = Random.new().read(AES.block_size)
enc = AES.new(key, AES.MODE_CBC, iv)
flag = b64encode(iv + enc.encrypt(pad(b'[REDACTED]', AES.block_size))).decode()
```

The things that stick out most obviously here is that a) python random isn't seeded, but it is used a _lot_ in the formation of the "spicy" version of the image and b) that same random function is used for the generation of the _key_, even when a perfectly good Crypto.Random is being used for the IV. Something that also sticks out to me is that, since this is a standard CBC implementation of AES, not having the IV is not a dealbreaker. The first plaintext block is XOR'd with the IV, with the rest of the ciphertext happily chugging away in parallel, correctly decrypting. As long as we can retrieve the key, we get most, if not all, of the plaintext.

As for key retrieval, by the time I looked into how python implements RNG by default, I knew I had actually solved this problem previously when going through the Cryptopals series of problems (they're great! https://cryptopals.com/).
### Solution
Python's default RNG is the **Mersenne Twister**, seeded with either urandom or, if not available, the datetime. For a split second, I had hoped I could get out easy by getting the datetime from the files and using that as the seed, but however Python seeds seems to obfuscate taht part in the source, and even then there's no guarentee that would get you the seed necessary.

However, when looking through python documentation (or by doing a cryptopals problem _on_ the MT), you'll find that it's ***extremely*** discouraged for cryptographic implementations, as you can reconstruct the internal state from 624 calls of the RNG (plus a couple for comfirmation). 

I _could_ hand code the untempering and such, but why bother when someone else has done it twelve times better? (https://github.com/eboda/mersenne-twister-recover)

All that's left now is reconstructing the state from the first 1000 or so calls, confirming that it matches with the remaining 1.8 million calls for the rest of the pixels, and getting the key at the end! The only sticky part left is, when retrieving the four bytes from each pixel, it's not clear whether random spat them out in big or little endian. Thankfully, the MT recover program yells at you for having an invalid state when using big endian integers, so that problem was solved easily enough.

### Solution script

```py
import random
from PIL import Image
from Crypto.Cipher import AES
from base64 import b64decode

boring = Image.open('Art_Final_2022.png', 'r').convert('RGBA')
boring_pix = boring.load()

spicy = Image.open('ENHANCED_Final_2022.png', 'r').convert('RGBA')
spicy_pix = spicy.load()

randomlist = list()
for i in range(1000):
    x = i % boring.size[0]
    y = i // boring.size[0]
    rgba = bytearray([bore ^ spice for bore, spice in zip(boring_pix[x, y], spicy_pix[x,y])])
    randomlist += [int.from_bytes(rgba,'little')] #originally used 'big', but MTRecover was unhappy with it, and very happy with 'little'

#insert missing code where i took the random list, used it in the reconstructor, and got the state back out, ctrl+v'ing into here.

#random.setstate(   insert random.getstate here (it's 624 integers, good lord)   )
for i in range(1000, boring.size[0] * boring.size[1]):
    x = i % boring.size[0]
    y = i // boring.size[0]
    rgba = bytearray([bore ^ spice for bore, spice in zip(boring_pix[x, y], spicy_pix[x,y])])
    assert rgba == random.randbytes(4)

#the fact that we got here is a miracle, really.
key = bytes(random.sample(random.randbytes(16), 16))
iv = b'\x00'\*16 #who cares
dec = AES.new(key, AES.MODE_CBC, iv)
flag = dec.decrypt(b64decode("Tl5nK8L2KYZRCJCqLF7TbgKLgy1vIkH+KIAJv5/ILFoC+llemcmoLmCQYkiOrJ/orOOV+lwX+cVh+pwE5mtx6w=="))
print(flag)

# b'b\x86(\xbc3\xab\x05\x8e\x08b\xf8\xe3\xaa\xdc\xbf\xd8vsctf{1_gu355_R4ND0m_i5nt_tH4T_5p1cy}\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b\x0b'
```

Flag: **vsctf{1_gu355_R4ND0m_i5nt_tH4T_5p1cy}**
