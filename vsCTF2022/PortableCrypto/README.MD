# Portable Crypto

### Description
I keep getting complaints about file size, so I made an encryption scheme using only a few characters!

### Exploration
_crypto.bf_
```
       "NOITPYRCNE GALF ELBATROP SV">:v
$                v                  |,<
v4p10+1:g10:_ #!\?+# <18421<p100,*25<
>p~+98*+"¢"%"Δ"+#^,01g66*%#^_@
(insert unholy unprintable characters here)
```
_out.txt_
```
VS PORTABLE FLAG ENCRYPTION
ξέΠαΣεΝПЪвЭΡΫΥТμеЬаГΧιМИΟФΜήГΰбНЭЏЃκ
```

When I first downloaded the files, I was tipped off at first by the file extension .bf, because my first thought was "oh cool, a Brainfuck challenge!" I've spent a decent amount of time messing around with Brainfuck, so I was already thinking about esoteric programming languages when I opened it up. As it clearly wasn't Brainfuck, however, I noticed this block of characters:
```
>:v
|,<
```
It looks suspiciously like arrows going in a circle, and if there's any esoteric programming language I know that has 2D arrows going in a circle, it's BeFunge! (https://esolangs.org/wiki/Befunge) Befunge is another language that was designed to be hell on whoever would work on backend things for it, except it was about building any compiler at all, instead of the smallest one in the case of Brainfuck. In a word, Befunge is 2D - designed to have a pointer on the code that moves around based on arrows that it encounters, conditional pipes, and, in the case of ?, _just picking a direction at random_. After finding a Befunge interpreter online (which actually doesn't work correctly - we'll get to that), I stepped through the Befunge to figure out exactly what it does
### Solution


### Solution script

Flag: **vsctf{1_gu355_R4ND0m_i5nt_tH4T_5p1cy}**
