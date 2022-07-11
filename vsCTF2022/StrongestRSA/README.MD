# Strongest RSA

### Description
I think `getStrongPrime` is not safe enough, so I implemented an RSA algorithm using the strongest primes ever. I don't think it's breakable unless you have a quantum computer!

### Exploration
```py
def getStrongestPrime(nbits):
    while True:
        p = getStrongPrime(nbits)
        delta = random.randint(0x1337, 0x1337 + 0x1337)
        pp = p - delta
        ppp = prevprime(factorial(pp) % p)
        if gcd(ppp-1, e) == 1:
            return p, ppp
    
NBITS = 1024
p0, p = getStrongestPrime(NBITS)
q0, q = getStrongestPrime(NBITS)
N = p * q
m = bytes_to_long(FLAG.encode())
c = pow(m, e, N)
```

Appears to be a textbook RSA implementation with long enough primes that throwing **n** into a factorizer will yield no results. As with many other RSA CTFs, this appears to come down to the way that the primes are generated - if you can find a reasonable flaw in the prime generation, the factorization will soon follow.

This prime generation looks something like "find the largest prime **ppp** such that **ppp < (p - delta)! % p**". We are also provided p (named **p0**). At first glance, this seems rather difficult - calculating the factorial of a 1024-bit prime (minus some piddly delta) will require 2^1024 multiplications or so, and we have 24 hours. Not exactly ideal. However, thanks to some number theory involving primes and factorials, we can work our way down instead of up.

### Solution
**Wilson's Theorem** (https://en.wikipedia.org/wiki/Wilson%27s_theorem) states that **(p-1)! = -1 mod p** when **p** is a prime. If I wasn't aware of this fact _before_ starting this problem, I'm not convinced I would have been able to find references to it during the challenge - searching "factorial of a prime" only comes up with Factorial primes, which are interesting in their own right, but useless for this. Perhaps the expectation is that any budding number theorist / cryptowhiz should know this? Who knows!

Regardless, this tool instantly gives us a shortcut to starting a _lot_ closer to the goal, and we only need to work backwards, at most, 2\*0x1337 iterations to find the relevant factorial value and, thus, the prime associated with it.

The plan is, then, to start with knowing **(p0-1)! = p0-1 mod p0**, and then working our way _down_ until we hit a prime that divides into N. To work down a step, simply "divide" (multiply by the inverse) of the current factorial argument, and decrement the factorial argument. So, you can get **(p0-2)! = (p0-1 \* pow(p0-1,-1,p0))%p0**, and so on, checking each value between **(p0-0x1337)!** and **(p0-2\*0x1337)!** for the previous prime and its candidacy for divisibility. I ended up using q0 in the script, as it resolved a lot quicker.

### Solution script

```py
from sympy import prevprime
#i'm so impatient
from tqdm import tqdm

p0 = 163753477176210014003355280732229891908166074468271556144642666169325605017666799921295576722168608401188682320182653287668989748162506955989407213845500704903463544753049275828138559289189335596749709834289278256382427251831790026921563375111737350084174473833546767952081017613072491759534988253353621530923
q0 = 157598184809589313845990455272198459548591786211953253450211152128535343234857067521711590445365424087430728267491317690639227988484930088637483194045435135802590588269993794073236513557034321374876808546159597997280236993358749182432517011554239468502233558179815446959403076134284375214662245037202945590183
N = 11884142558095727641000594156833818117849240126500615037738361957005811068956622520280143210434649198031005585252791693777710458190732464123269660559382653636999601459113099276826723072914352276709761755328542359490331355061792823458149611674845846523699218971126655186522340818792078719216860046464292413878045842425132308544311887062610272360069819975798905665533964761527225558339025724872067751916657135473510775709503714808686565298632040214249698116863336246844759838665285888816202570667521796553678688293761589082062045634768520102235077364345013564344229095323239077977717497503322831684471959195555281580807
e = 65537
c = 11776079752956619284016871274992903352398310565005810097721997339193718454945819135683541554652454321040530044545154341786048659896370226535387839157317585368391189570502841702311449000698372030666509296004039398083488490698999338894328619127149024309470011330855840757405205104944658961386764569043610715311746676861275270073394069269043429092551681704290340091149637137627751767730812255069347108706434972786681985484368054390699974613090342753508097177008167140924577095976699437810398922852319420301082587264411993737330188227703869101718515748828944300463051133118636928879090217708121368293440440444106196607645

#thanks to wilson's theorem, we start here!
#i ended up using q because p was too slow - this program chugs if the prime isn't found fast enough
# a, (n-a)! % p
fac_pair = (1, q0-1)

#for going back a step in the factorial
def undo(pair, p):
    return (pair[0] + 1, (pow(p-pair[0], -1, p) * pair[1]) % p)

while fac_pair[0] < 0x1337:
    fac_pair = undo(fac_pair, q0)

print("initial part done - on to scanning")

final_p = 1
for i in tqdm(range(0x1337, 0x1337+0x1337)):
    if N % prevprime(fac_pair[1]) == 0:
        final_p = prevprime(fac_pair[1])
        break
    fac_pair = undo(fac_pair, p0)

if final_q != 1:
    print("found the prime!! q = {}".format(final_q))
    final_p = N // final_q
    phiN = (final_p-1)*(final_q-1)
    d = pow(e,-1,phiN)
    pt = pow(c,d,N)
    print(pt.to_bytes(128,'big'))
 
#initial part done - on to scanning
#(tqdm nonsense deleted)
#found the prime!! q = 98012277990559371582983072655226283623673957669823428474469843558976683611088649921733473848543984940795021747022803821693331071948319773554362412970452779722021196926620622353412479180070638846582718441833102198044546885761608566645177571964681930259939794787139725829793461617386332420481477517399921209143
#b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00vsctf{Strongest_can_be_the_weakest:(}'
```

Note: Almost first blooded this one. It really seemed like I did, but I was beat by seconds by **willwam845**. According to the vsCTF info on them, this was the only challenge they solved. Hats off for such a chad move.

Flag: **vsctf{Strongest_can_be_the_weakest:(}**
