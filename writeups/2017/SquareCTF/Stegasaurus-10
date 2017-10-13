# Stegasaurus

## Description
We captured this image from an old database filled with messages from the first robot spies. Most of these took the form of dinosaur or dragon action figures. This one was sent in by a dinosaur bot, with the note "Stegasauruses are like Onions: they have layers, and like hiding in grass. Replace spaces with dashes and prepend "flag-" to the flag."
https://cdn.squarectf.com/challenges/stegasaurus.png

### Solution

This is a simple Steganography challenge. we are given a png file which is just a square shape (Square's logo)

Initially I loaded in in GIMP but it didn't yield any results.

I fired zsteg with the filename:

```
# zsteg stegasaurus.png 
b1,r,msb,xy         .. text: "import sys\nk=int(sys.argv[1]);print(\"\".join(chr(ord(c)^k) for c in \"Fpvgpa8'M,m&!s,!\"))"
b4,r,lsb,xy         .. text: "GTTUETDDUTU{/"
b4,rgb,lsb,xy       .. file: MPEG ADTS, layer II, v1, 256 kbps, Monaural
b4,rgb,msb,xy       .. text: "*\"\"\"\"\"\"\""
```

Interesting, we see a one-liner Python which takes an integer as argument. this is the prettified code:

```
import sys
k=int(sys.argv[1])
for c in "Fpvgpa8'M,m&!s,!":
    print("\"".join(chr(ord(c)^k)))
```

Passing arguments to it results in the following:

```
root@toxicity:/tmp# python steg.py 1
G
q
w
f
q
`
9
&
L
-
l
'
 
r
-


root@toxicity:/tmp# python steg.py 1000
Traceback (most recent call last):
  File "t", line 4, in <module>
    print("\"".join(chr(ord(c)^k)))
ValueError: chr() arg not in range(256)

```
Ok so this should be easy, we only have 250~ options to go through. we can easily brute force.

I slightly modified the Python to print all characters on the same line.

```
chars=[]
for k in range(50):
  for c in "Fpvgpa8'M,m&!s,!":
    chars.append("\"".join(chr(ord(c)^k)))
  print("".join(str(x) for x in chars))
  del chars[:]
```

and then ran it again

```
root@toxicity:/tmp# python steg.py
Iyhn7(B#b).|#.
V`fw`q(7]<}61c<1
Wagvap)6\=|70b=0
Tbdubs*5_>43a>3
Ucetcr+4^?~52`?2
Rdbsdu,3Y8y25g85
Secret-2X9x34f94
Pf`qfw.1[:{07e:7
Qgapgv/0Z;z16d;6
^hnhy ?U4u>9k49
_io~ix!>T5t?8j58
\jl}j{"=W6w<;i6;
]km|kz#<V7v=:h7:
Zlj{l}$;Q0q:=o0=
[mkzm|%:P1p;<n1<
Xnhyn&9S2s8?m2?
Yoixo~'8R3r9>l3>

```

Output seems like garbage, except one line: Secret-2X9x34f94

flag: flag-Secret-2X9x34f94
