# adventures of a kid that tries to transfer files thru SMS
or, the methodology of sms-file-transfer (SFT).

## Why?
~~... idk really 🤷‍♂️~~ <br>
*The goal is to convert/compress a [Base64 string (usually used by files on the web)](https://en.wikipedia.org/wiki/Base64) to a [GSM 03.38-encoded string (used in SMS)](https://en.wikipedia.org/wiki/GSM_03.38) for exchanging files over SMS.*

<hr>

Throughout this file, we'll be converting this Base64 string:
```
data:image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA
```

## Preface
We will need to use the entire available character set in order to get the maximum efficiency of the conversion.
```
@£$¥èéùìòÇØøÅåΔ_ΦΓΛΩΠΨΣΘΞÆæßÉ!"#¤%&'()*+,-./0123456789:;<=>?¡ABCDEFGHIJKLMNOPQRSTUVWXYZÄÖÑÜ§¿abcdefghijklmnopqrstuvwxyzäöñüà
```
We can pick the first 120 characters for our custom Base120 implementation and the last 4 as a "special character set".


### Base120 characters
These are the set of characters we will use to convert Base64 characters into.
<details>
  <summary>Base120 characters and and some equivalent bases</summary>

Letter | Decimal equivalent | Base3 equivalent (will use later)
--- | --- | ---
@ | 000 | 00000
£ | 001 | 00001
$ | 002 | 00002
¥ | 003 | 00010
è | 004 | 00011
é | 005 | 00012
ù | 006 | 00020
ì | 007 | 00021
ò | 008 | 00022
Ç | 009 | 00100
Ø | 010 | 00101
ø | 011 | 00102
Å | 012 | 00110
å | 013 | 00111
Δ | 014 | 00112
_ | 015 | 00120
Φ | 016 | 00121
Γ | 017 | 00122
Λ | 018 | 00200
Ω | 019 | 00201
Π | 020 | 00202
Ψ | 021 | 00210
Σ | 022 | 00211
Θ | 023 | 00212
Ξ | 024 | 00220
Æ | 025 | 00221
æ | 026 | 00222
ß | 027 | 01000
É | 028 | 01001
! | 029 | 01002
" | 030 | 01010
\# | 031 | 01011
¤ | 032 | 01012
% | 033 | 01020
& | 034 | 01021
' | 035 | 01022
( | 036 | 01100
) | 037 | 01101
\* | 038 | 01102
\+ | 039 | 01110
, | 040 | 01111
\- | 041 | 01112
. | 042 | 01120
/ | 043 | 01121
0 | 044 | 01122
1 | 045 | 01200
2 | 046 | 01201
3 | 047 | 01202
4 | 048 | 01210
5 | 049 | 01211
6 | 050 | 01212
7 | 051 | 01220
8 | 052 | 01221
9 | 053 | 01222
: | 054 | 02000
; | 055 | 02001
< | 056 | 02002
= | 057 | 02010
\> | 058 | 02011
? | 059 | 02012
¡ | 060 | 02020
A | 061 | 02021
B | 062 | 02022
C | 063 | 02100
D | 064 | 02101
E | 065 | 02102
F | 066 | 02110
G | 067 | 02111
H | 068 | 02112
I | 069 | 02120
J | 070 | 02121
K | 071 | 02122
L | 072 | 02200
M | 073 | 02201
N | 074 | 02202
O | 075 | 02210
P | 076 | 02211
Q | 077 | 02212
R | 078 | 02220
S | 079 | 02221
T | 080 | 02222
U | 081 | 10000
V | 082 | 10001
W | 083 | 10002
X | 084 | 10010
Y | 085 | 10011
Z | 086 | 10012
Ä | 087 | 10020
Ö | 088 | 10021
Ñ | 089 | 10022
Ü | 090 | 10100
§ | 091 | 10101
¿ | 092 | 10102
a | 093 | 10110
b | 094 | 10111
c | 095 | 10112
d | 096 | 10120
e | 097 | 10121
f | 098 | 10122
g | 099 | 10200
h | 100 | 10201
i | 101 | 10202
j | 102 | 10210
k | 103 | 10211
l | 104 | 10212
m | 105 | 10220
n | 106 | 10221
o | 107 | 10222
p | 108 | 11000
q | 109 | 11001
r | 110 | 11002
s | 111 | 11010
t | 112 | 11011
u | 113 | 11012
v | 114 | 11020
w | 115 | 11021
x | 116 | 11022
y | 117 | 11100
z | 118 | 11101
ä | 119 | 11102
</details>

### Special character set
Special characters |
-- |
ö |
ñ |
ü |
à |

These characters are "reserved" and won't be used in converting Base64 characters.

## Step 0
A sort-of 'version indentifier' for the compressed string so that old and new SFT decoders knows if the strings are compatible to them or not. Placed at the very first part of the string.

**Current version: `1.0` <br>**
**Character assigned to current version: `@`**

⚠ The characters must not be any special characters so as to differentiate it from other steps (particularly Step 2).

## Step 1
```
┌───┐
data:image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA
```
Since the `data:` part is present in every single Base64 string, we can just remove it.

## Step 2
```
     ┌────┐
data:image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA

encoded so far:
@image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA (97.18%)
```

The commonly used strings in this part is `image/` and `application/`, so we can just replace it with a corresponding special character.

Special Character | replaces
-- | --
à | `image/`
ü | `application/`
No special char | Don't replace

### Why not the whole `image/bmp;`?
Encoder and decoder will be too big by then.

## Step 3
```
               ┌─────┐
data:image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA

converted so far:
@à/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA (94.37%)
```

Basically the same as Step 1; Since the `base64,` part is present if every single Base64 string, we can just remove it.

## Step 4
*the actually interesting part! (which is split into several parts)*
```
                      ┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
data:image/bmp;base64,Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA

converted so far:
@à/bmp;Qk1aAAAAAAAAAD4AAAAoAAAACwAAAAcAAAABAAEAAAAAAAAAAADDDgAAww4AAAIAAAACAAAA/////wAAAP8AAAAAbsAAACpAAABowAAAQIAAAGDAAAAAAAAA (89.44%)
```

### Part 1
*probably the easiest compression algorithm*

Basically, `AAAAAABCCCDD` becomes `A5BC2DD`. But instead of numbers, we'll be using... you guessed it, our special character set.

```
 à   00   1
 ü   01   2
 ñ   02   3
 ö   03   4
üà   10   5
üü   11   6
üñ   10   7
     ...
```
so, `AAAAAABCCCD` becomes `AüàBCüDD`.

```
converted so far:
@à/bmp;Qk1aAñàD4AöoAöCwAöcAöBAüEAññDñgAüwü4AñIAöCAö/üàwAñP8AüàbsAñCpAñBowAñQIAñGDAñà (59.15%)
```

### Part 2
Converting Base64 to Base120

Soon™

## Epilogue

## Future Improvements

## Links
