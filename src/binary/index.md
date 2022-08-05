---
title: "Binary Storage"
syllabus:
- FIXME
---

Python and other high-level languages shield programmers from low-level details,
but sooner or later someone has to worry about bits and bytes.
This chapter explores computers represent numbers and text
and how to work with binary data.

## Integers {: #binary-storage-int}

Let's start by looking at how integers are stored.
The natural way to do this with ones and zeroes is to use base 2,
so 1001 in binary is (1×8)+(0×4)+(0×2)+(1×1) or 9 base 10.
We can handle negative numbers by reserving the top bit for the sign,
so that 01001 is +9 and 11001 is -9.

This representation has two drawbacks.
The minor one is that it gives us two zeroes,
one positive and one negative.
More importantly,
the hardware needed to do arithmetic
on this [%g sign_magnitude "sign and magnitude" %] representation
is more complicated than the hardware needed for another scheme
called {% g twos_complement "two's complement" %].
Instead of mirroring positive values,
two's complement rolls over when going below zero like an odometer.
For example,
with three-bit integers we get the values in [%t binary-storage-3bit %]

<div class="table" id="binary-storage-3bit" caption="3-bit integer values" markdown="1">
| Base 10 | Base 2 |
| ------- | ------ |
| 3       | 011    |
| 2       | 010    |
| 1       | 001    |
| 0       | 000    |
| -1      | 111    |
| -2      | 110    |
| -3      | 101    |
| -4      | 100    |
</div>

We can still tell whether a number is positive or negative
by looking at the first bit:
negative numbers have a 1, positives have a 0.
However,
two's complement is asymmetric:
since 0 counts as a positive number,
numbers go from -4 to 3, or -16 to 15, and so on.
As a result,
even if `x` is a valid number,
`-x` may not be.

### Writing Binary

We can write binary numbers directly in Python by using the `0b` prefix.
For example, `0b0011` is the number 3.
Programmers usually use [%g hexadecimal "hexadecimal" %] (base 16) instead:
the digits 0–9 have the usual meaning,
and the letters A-F (or a-f) are used to represent the numbers 11–15.
We signal that we're using hexadecimal with a `0x` prefix,
so `0xF7` is (15×16)+7 or 247.

Each hexadecimal digit corresponds to four bits ([%t binary-hex %]),
which makes it easy to translate bits to digits and vice versa:
for example,
`0xF7` is `0b11110111`.
As a bonus,
two hexadecimal digits is exactly one byte.

<div class="table" id="binary-hex" caption="Bitwise operations" markdown="1">
| Decimal | Hexadecimal | Bits |
| ------- | ----------- | ---- |
| 0       | 0           | 0000 |
| 1       | 1           | 0001 |
| 2       | 2           | 0010 |
| 3       | 3           | 0011 |
| 4       | 4           | 0100 |
| 5       | 5           | 0101 |
| 6       | 6           | 0110 |
| 7       | 7           | 0111 |
| 8       | 8           | 1000 |
| 9       | 9           | 1001 |
| 10      | A           | 1010 |
| 11      | B           | 1011 |
| 12      | C           | 1100 |
| 13      | D           | 1101 |
| 14      | E           | 1110 |
| 15      | F           | 1111 |
</div>

## Bitwise Operations {: #binary-ops}

Like most languages based on C,
Python provides four operators for working with bits:
`&` (and),
`|` (or),
`^` (xor),
and `~` (not).
`&` yields a 1 only if both its arguments are 1's,
while `|` yields 1 if either or both of its arguments are 1.
`^`, called [%g exclusive_or "exclusive or" %] or [%g xor "xor" %],
produces 1 if either but *not* both of its arguments are 1.
Putting it another way,
`^` produces 0 if its inputs are the same and 1 if they are different.
Finally,
`~` flips its argument: 1 becomes 0, and 0 becomes 1.
When these operators are used on multibit values
they work on corresponding bits independently as shown in [%t binary-ops %].

<div class="table" id="binary-ops" caption="Bitwise operations" markdown="1">
| Expression | Bitwise       | Result          |
| ---------- | ------------- | --------------- |
| `12 & 6`   | `1100 & 0110` | `4` (`0100`)    |
| `12 | 6`   | `1100 | 0110` | `14` (`1110`)   |
| `12 ^ 6`   | `1100 ^ 0110` | `10` (`1010`)   |
| `~ 6`      | `~ 0110`      | `9` (`1001`)    |
| `12 << 2`  | `1100 << 2`   | `48` (`110000`) |
| `12 >> 2`  | `1100 >> 2`   | `3` (`0011`)    |
</div>

We can set or clear individual bits with these operators.
To set a particular bit,
create a value in which that bit is 1 and the rest are 0.
When this is or'd with a value,
the bit we set is guaranteed to come out 1;
the other bits will be left as they are.
Similarly,
to  set a bit to zero,
create a value in which that bit is 0 and the others are 1,
then use `&` to combine the two.
To make things easier to read,
programmers often set a single bit,
negative it with `~`,
and then use `&`:

[% excerpt f="bit_mask.py" %]

Most C-inspired languages also provide [%g bit_shift "bit shifting" %] operators.
that move bits left or right.
Shifting the bits `0110` left by one place produces `1100`,
while shifting it right by one place produces `0011`.
In Python,
this is written `x << 1` or `x >> 1`.

Just as shifting a decimal number left corresponds to multiplying by 10,
shifting a binary number left is the same as multiplying it by 2.
Similarly,
shifting a number right corresponds to dividing by 2 and throwing away the remainder,
so `17 >> 3` is 2.

But what if the top bit of an integer changes from 1 to 0 or vice versa as a result of shifting?
If we're using two's complement,
then the bits `1111` represent the value -1;
if we shift right we get `0111` which is 7.
Similarly,
if we shift `0111` to the left we get `1110` (assuming we fill in the bottom with 0),
which is -2.

Different languages deal with this problem in different ways.
Python always fills with zeroes.
Java provides two versions of right shift:
`>>` fills in the high end with zeroes
while `>>>` copies in the topmost (sign) bit of the original value.
C (and by extension C++) lets the underlying hardware decide,
which means that if you want to be sure of getting a particular answer
you have to handle the top bit yourself.

## Text {: #binary-storage-text}

The rules for storing text make integers look simple.
By the early 1970s most programs used [%g ascii "ASCII" %],
which represented unaccented Latin characters using the numbers from 32 to 127.
(The numbers 0 to 31 were used for [%g control_code "control codes" %]
such as newline, carriage return, and bell.)
Since computers use 8-bit bytes and the numbers 0–127 only need 7 bits,
programmers were free to use the numbers 128–255 for other characters.
Unfortunately,
different pieces of software used them to represent different symbols:
non-Latin characters,
graphic characters like boxes,
and so on.
The chaos was eventually tamed by the [%g ansi_character "ANSI" %] standard
which (for example) defined the value 231 to mean the character "ç".

But the ANSI standard only solved part of the problem.
The ANSI standard didn't include characters from Turkish, Devanagari, and many other alphabets,
much less the thousands of characters used in some East Asian writing systems.
One solution would have been to use 16 or even 32 bits per character,
but:

1.  existing text files using ANSI would have to be transcribed, and
2.  documents would be two or four times larger.

The solution was a new standard called [%g unicode Unicode %] with two parts.
The first part defined a [%g code_point "code point" %] for every character:
U+0065 for an upper-case Latin "A",
U+2605 for a black star,
and so on.
The second part defined ways to store these values in memory.
The simplest of these is [%g utf_32 "UTF-32" %],
which stores every character as a 32-bit number.
This wastes a lot of memory—if the text is written in a Western European language,
UTF-32 uses four times as much storage as necessary—but
since each character is exactly the same size it's very easy to process.

The most popular encoding is [%g utf_8 "UTF-8" %],
which is a [%g variable_length_encoding "variable length encoding" %].
Every code point from 0 to 127 is stored in a single byte whose high bit is 0,
just as it was in the original ASCII standard.

What if the high bit in the byte is 1?
In that case,
the number of 1's after the high bit but before the first 0
tells UTF-8 how many more bytes that character is using.
For example,
if the first byte of the character is `11101101` then:

-   the first 1 signals that this is a multi-byte character;
-   the next two 1's signal that the character includes bits
    from the following two bytes as well as this one;
-   the 0 separates the byte count from the first few bits used in the character;
    and
-   the final 1101 is the first four bits of the character.

But that's not all:
every byte that's a continuation of a character starts with 10.
This rule means that if we look at any byte in a string
we can immediately tell if it's the start of a character
or the continuation of a character.
Thus,
if we want to represent a character whose code point is 1789:

-   We convert to binary 11011111101.
-   We count and realize that we'll need two bytes:
    the first storing the high 5 bits of the character,
    the second storing the low 6 bits.
-   We encode the high 5 bits as 11011011,
    meaning "start of a character with one continuation byte
    and the 5 payload bits 11011".
-   We encode the low 6 bits as 10111101,
    meaning "a continuation byte wiht 6 payload bits 111101".

## Floating Point Numbers {: #binary-storage-fp}

The rules for floating point numbers make Unicode look simple.
The root of the problem is that
we cannot represent an infinite number of real values
with a finite set of bit patterns.
And no matter what values we represent,
there will be an infinite number of values between each of them that we can't.

<div class="callout" markdown="1">

### Go to the source

The explanation that follows is simplified to keep it manageable.
If you're doing any calculation on a computer at all,
please read [%b Goldberg1991 %].

</div>

Floating point numbers are represented by a sign,
a magnitude,
and an exponent.
In a 32-bit [%g word_memory "word" %]
the IEEE 754 standard calls for 1 bit of sign,
23 bits for the magnitude (or mantissa),
and 8 bits for the exponent.
We will illustrate how it works using a much smaller representation:
no sign,
3 bits for the magnitude,
and 2 for the exponent.

[% fixme "represent 48" $]

Here are the values we can represent this way:

[% fixme "number line" %]

There is a lot of redundancy here
which the IEEE standard avoids by shifting things around.
More importantly,
this format can't represent a lot of values:
for example,
it can store 8 and 10 but not 9.
This is exactly like the problem hand calculators have
with fractions like 1/3:
in decimal, we have to round that to 0.3333 or 0.3334.

But if this scheme has no representation for 9
then 8+1 must be stored as either 8 or 10.
If that's so,
then what is 8+1+1?
If we add from the left,
(8+1)+1 is 8+1 is 8,
but if we add from the right,
8+(1+1) is 8+2 is 10.
Changing the order of operations makes the difference between right and wrong.

The authors of numerical libraries spend a lot of time worrying about things like this.
In this case
sorting the values and then add from smallest to largest
gives the best chance of getting the best possible answer.
In other situations,
like inverting a matrix, the rules are much more complicated.

Another observation about our uneven number line is that
the we can represent are unevenly spaced.
However,
the *relative* spacing between each set of values stays the same:
the first group is separated by 1,
then the separation becomes 2,
then 4,
and so on.
This points us at a couple of useful definitions:

-   The [%g absolute_error "absolute error" %] in an approximation
    is the absolute value of the difference
    between the approximation and the actual value.

-   The [%g relative_error "relative error" %]
    is the ratio of the absolute error
    to the absolute value we're approximating.

For example,
being off by 1 in approximating 8+1 and 56+1 is the same absolute error,
but the relative error is larger in the first case than in the second.
{: .continue}

Relative error is almost always more useful than absolute:
it makes little sense to say that we're off by a hundredth
when the value in question is a billionth.
We should take this into account when testing.
The program shown below loops over the integers from 1 to 9
and uses them to create the values 0.9, 0.09, and so on:

[% fixme "program to calculate numbers" %]

It then sums those numbers to produce 0.9, 0.99, and so on.

Let's calculate the same values by subtracting .1 from 1,
then subtracting .01,
and so on.
This program does that and computes the difference
between the two representations:

The very first value contributing to our sum is already slightly off
because we cannot exactly represent 0.9 in base 2
any more than we can exactly represent 1/3 in base 10.
Doubling the size of the mantissa would reduce the error,
but we can't ever eliminate it.
The good news is that 9×0.1 and 1-0.1 are exactly the same.
It might not be precisely right, but at least it's consistent.

The same cannot be said for some of the later values:
there are small differences between what we get from our two formulas.
And sometimes the accumulated errors cancel out
and make the result more accurate once again.

One implication of this is that
we should never compare floating point numbers with `==` or `!=`
because two numbers calculated in different ways
will probably not have exactly the same bits.
It's safe to use `<`, `>=`, and other orderings,
though,
since they don't depend on being the same down to the last bit.

If we do want to compare floating point numbers
we can use something like [the `approx` class][pytest_approx] from [pytest][pytest]
which checks whether two numbers are within some tolerance of each other.
A completely different approach is to use something like
[the `fractions` package][python_fractions],
which (as its name suggests) uses numerators and denominators
to avoid some precision issues.
[This post][textualize_fraction] describes one clever use of the package.

## And Now, Persistence {: #binary-storage-binary}

So why store data in a format that can't be edited with Notepad or nano?
There are generally four reasons:

Size
:   The string `"10239472"` is 8 bytes long,
    but the 32-bit integer it represents only needs 4 bytes in memory.
    This doesn't matter for small data sets,
    but it does for large ones,
    and it definitely does when data has to move between disk and memory
    or between different computers.

Speed
:   Adding the integers 34 and 56 is a single machine operation.
    Adding the values represented by the strings `"34"` and `"56"` is dozens;
    we'll explore this in the exercises.
    Most programs that read and write text files
    convert the values in those files into binary data
    using something like the `int` or `float` functions,
    but if we're going to process the data many times,
    it makes sense to avoid paying the conversion cost over and over.

Hardware
:   Someone, somewhere, has to convert the signal from the thermocouple to a number,
    and that signal probably arrives arrives as a stream of 1's and 0's.
    
Lack of anything better
:   It's possible to represent images as ASCII art, but sound?
    Or video?
    It would be possible, but it would hardly be sensible.

Most programs use line-oriented file I/O:
they read characters until they see an end-of-line marker
and then hand back those characters as a string.
We can also use byte-oriented routines,
the most basic of which is simply called `read`.
If `stream` is an open file,
then `stream.read(N)` hands back up to the next N bytes from the file
("up to", because there might not be that much data left).
The result is returned as a string,
but---and this is crucial---there is no guarantee that the values represent characters.
We can concatenate other data onto it,
but if the underlying file is a PNG image,
text-oriented methods like `string.upper`
won't do anything meaningful.

[% fixme "rewrite this to explain bytes instead of strings" %]

Where there's a `read` there's a `write`.
`stream.write(str)` writes the bytes in the string `str` to a file that has been opened for writing.
In both the reading and writing cases,
though,
it's very important to open the file in [%g binary_mode "binary mode" %] using either:

```python
reader = open('input.dat', 'rb')
```

or:
{: .continue}

```python
writer = open('input.dat', 'wb')
```

The `"b"` at the end of the mode string tells Python
*not* to translate Windows line endings (which are the two characters `"\r\n"`)
into Unix line endings (the single character `"\n"`).
This translation is handy when we're working with text,
since it means our programs only have to deal with one style of line
ending no mater what platform the code is running on,
but it messes up non-textual data.

There's another problem here as well.
C and Fortran store integers as "naked" 32-bit values:
the program uses what the machine provides,
no more and no less.
Python and other dynamic languages usually don't use raw values.
Instead,
they put the value in a larger data structure
that keeps track of its type along with a bit of extra administrative information.
That extra data allows those languages to do garbage collection.
It also allows us to assign values to variables without explicitly declaring their type,
since the value we're assigning carries its type along with it.

A similar issue comes up when we compare Fortran's arrays to Python's lists.
Fortran stores the data in an array side by side in one big block of memory.
Writing this to disk is easy:
if the array starts at location L in memory and has N values,
each of which is B bytes long,
we just copy the bytes from L to L+NB-1 to the file.

A Python list,
on the other hand,
stores pointers to values rather than the values themselves.
To put the values in a file
we can either write them one at a time
or pack them into a contiguous block and write that.
Similarly,
when reading from a file,
we can either grab the values one by one
or read a larger block and then unpack it in memory.

Packing data is a lot like formatting values for textual output.
The format specifies what types of data are being packed,
how big they are (e.g., is this a 32-bit or 64-bit floating point number?),
and how many values there are.
The format exactly determines how much memory is required by the packed representation.
The result of packing values is a block of bytes,
which Python represents as a string,
but as mentioned above,
this isn't a string of characters.

Unpacking reverses this process.
After reading data into memory
we can unpack it according to a format.
The most important thing is that
*we can unpack data any way we want*.
We might pack an integer and then unpack it as four characters,
since both are 32 bits long.
Or we might save two characters,
an integer,
and two more characters,
then unpack it as a 64-bit floating point number.
The bits are just bits:
it's our responsibility to make sure we keep track of their meaning
when they're down there on disk.

In Python we can use the `struct` module to pack and unpack data.
The function `pack(format, val_1, val_2, …)`
takes a format string and a bunch of values as arguments,
packs them into a string,
and gives that back to us.
The inverse function, `unpack(format, string)` takes such a string and a format
and returns a tuple containing the unpacked values.
Here's an example:

[% excerpt pat="simple_struct.*" fill="py out" %]

What is `\x1f` and why is it in our data?
If Python finds a character in a string that doesn't have a printable representation,
it prints a 2-digit escape sequence in [%g hexadecimal "hexadecimal" %] (base 16).
This uses the letters A-F (or a-f) to represent the digits from 10 to 15,
so that (for example) `3D5` is (3×16^2^)+(13×16^1^)+(5×16^0^), or 981 in decimal.
Python is therefore telling us that
our string contains the eight bytes
`['\x1f', '\x00', '\x00', '\x00', 'A', '\x00', '\x00', '\x00']`.
`1F` in hex is (1×16^1^)+(15×16^0^), or 31;
`'A'` is our 65,
because the ASCII code for an upper-case letter A is the decimal value 65.
All the other bytes are zeroes (`"\x00"`)
because each of our integers is 32 bits long
and the significant digits only fill one byte's worth of each.

<div class="table" id="binary-storage-formats" caption="`struct` package formats" markdown="1">
| Format | Meaning                                     |
|------- | ------------------------------------------- |
| `"c"`  | Single character (i.e., string of length 1) |
| `"B"`  | Unsigned 8-bit integer                      |
| `"h"`  | Short (16-bit) integer                      |
| `"i"`  | 32-bit integer                              |
| `"f"`  | 32-bit float                                |
| `"d"`  | Double-precision (64-bit) float             |
</div>

The `struct` module offers a lot of different formats,
some of which are shown in [%t binary-storage-formats %].
The `"B"`, `"h"`, and `"2"` formats deserve some explanation.
`"B"` takes the least significant 8 bits out of an integer and packs those;
`"h"` takes the least significant 16 bits and does likewise.
They're needed because binary data formats often store only as much data as they need to,
so we need a way to get 8- and 16-bit values out of files.
(Many audio formats,
for example,
only store 16 bits per sample.)

Any format can be preceded by a count,
so the format `"3i"` means "three integers":

[% excerpt pat="pack_count.*" fill="py out" %]

We get the wrong answer in the last call
because we only told Python to pack five characters.
How can we tell it to pack all the data that's there regardless of length?

The short answer is that we can't:
we must specify how much we want packed.
But that doesn't mean we can't handle variable-length strings;
it just means that we have to construct the format on the fly:

```python
format = f"{len(str)}s"
```

`len(str)` is just the length of the string `str`,
so if `str` contains the string `"example"`,
the expression above will assign the string `"7s"` to `format`,
which just happens to be exactly the right format to use to pack it.

That's fine when we're writing,
buthow do we know how much data to get if we're reading?
For example, suppose we have the two strings "hello" and "Python".
we can pack them like this:

```python
pack('5s6s', 'hello', 'Python')
```

but how do I know how to unpack 5 characters then 6?
The trick is to save the size along with the data.
If we always use exactly the same number of bytes to store the size,
we can read it back safely,
then use it to figure out how big our string is:
{: .continue}

```python
>>> def pack_string(str):
...     header = pack('i', len(str))
...     body_format = '%ds' % len(str)
...     body = pack(body_format, str)
...     return header + body
...
>>> pack_string('hello')
'\x05\x00\x00\x00hello'
```

The unpacking function is almost the same:

```python
>>> def unpack_string(buffer):
...    header, body = buffer[:4], buffer[4:]
...    unpacked_header = unpack('i', header)
...    length = unpacked_header[0]
...    body_format = '%ds' % length
...    result = unpack(body_format, body)
...    return result
```

First, we break the buffer into two parts:
a header that's exactly four bytes long
(i.e., the right size for an integer)
and a body made up of whatever's left.
We then unpack the header,
whose format we know,
to determine how many characters are in the string.
Once we've got that we use the trick shown earlier
to construct the right format on the fly
and then unpack the string and return it.

Something else to notice here is that
the least significant byte of an integer comes first.
This is called [%g little_endian "little-endian" %] and is used by all Intel processors.
Some other processors put the most significant byte first,
which is called [%g big_endian "big-endian" %].
There are pro's and con's to both, which we won't go into here.
What you *do* need to know is that if you move data from one architecture to another,
it's your responsibility to flip the bytes around,
because the machine doesn't know what the bytes mean.
This is such a pain that the `struct` library and other libraries like
will do things for you if you ask it to.
If you're using `struct`,
the first character of a format string optionally indicates the byte order
([%t binary-storage-endian %]).

<div class="table" id="binary-storage-endian" caption="`struct` package endian indicators" markdown="1">
| Character | Byte order | Size     | Alignment     |
| --------- | ---------- | -------- | ------------- |
| `@`       | native     | native   | native        |
| `=`       | native     | standard | none          |
| `<`       | little     | endian   | standard none |
| `>`       | big        | endian   | standard none |
| `!`       | network    | standard | none          |
</div>

You should also use the `struct` library's `calcsize` function,
which tells you how large (in bytes) the data produced or consumed by a format will be.
For example:

```python
>>> calcsize('4s')
4

>>> calcsize('3i4s5d')
56
```

Binary data is to programming what chemistry is to biology:
you don't want to spend any more time thinking at its level than you have to,
but there's no substitute when you *do* have to.
Please remember that libraries already exist to handle almost every binary format ever created
and to read data from almost every instrument on the market.
You shouldn't worry about 1's and 0's unless you really have to.

## Exercises {: #binary-storage-exercises}

FIXME

### Adding strings {: .exercise}

Write a function that takes two strings of digits
and adds them as if they were numbers
*without* actually converting them to numbers.
For example,
`add_str("12", "5")` should produce the string `"17"`.