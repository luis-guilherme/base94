Base94
######

A reversible translation of binary data to text with an alphabet between 2 and 94 symbols.

Description
===========

This is a binary/text converter with any base between 2 and 94. It is likely a theoretical prototype and carries a pure academical flavor as the time and space complexities make it applicable only on small files (up to a few tens of kilobytes), although it allows one to choose any base with no dependencies on powers of two, e.g. 7 or 77.

How to run
==========

.. code-block:: bash

    ./base94.py <-e|-d> src dst [base=42]

For example:

.. code-block:: bash

    ./base94.py -e /bin/gethostip /tmp/gethostip.b94 94
    ./base94.py -e /bin/gethostip /tmp/gethostip.b42
    ./base94.py -d /tmp/gethostip.b42 /tmp/gethostip

There is also ``test.sh`` script that uses a small binary file ``/bin/gethostip``
for encoding/decoding and gathering different metrics. Simply run it with no arguments.

Background
==========

The main purpose of such a converter is to put a binary file to a form applicable to send over a channel with a limited range of supported symbols. A good example is any text-based network protocol, like HTTP or SMTP, where all transmitted binary data has to be reversibly converted to a pure text form and having no control symbols as a part of such data. As it's known, the ASCII codes from 0 to 31 are considered to be control characters, and that's why they will be definitely lost while transmitting over any logical channel that doesn't allow endpoints to transmit full 8-bit bytes (binary) with codes from 0 to 255.

The standard solution nowadays for this purpose is the Base64 algorithm defined in the `RFC 4648`_ (easy to read). It also describes Base32 and Base16 as possible variations. The key point here: they all share the same characteristic as they are all powers of two. The wider a range of supported symbols (codes) the more space efficient result of conversion is. It will be bigger anyway, but the only question is how much bigger. For example, Base64 encoding gives an approximately 33% bigger output, because 3 input (8 valued bits) bytes are translated to 4 output (6 valued bits, 2^6=64) bytes. So, the ratio is always 4/3 that is the output is bigger by 1/3 or 33.(3)%. Practically speaking, Base32 is very inefficient because it implies translating 5 input (8 valued bits) bytes to 8 output (5 valued bits, 2^5=32) bytes and the ratio is 8/5, that is output is bigger by 3/5 or 60%. In this context, it is hard to consider any sort of efficiency of Base16 as its output size is bigger by 100% (each byte with 8 valued bits is represented by two 4 valued bits bytes, also know as the nibbles_, 2^4=16). It is not even a translation, but rather just a representation of an 8-bit byte in the hexadecimal view.

If you're curious, how these input/output byte ratios were calculated for the Base64/32/16 encodings, then the answer is the LCM (Least Common Multiple). Let's calculate them by ourselves, and for this, we'll need one more function, GCD (Greatest Common Divisor)

1. Base64 (Input: 8 bits, Output: 6 bits):
    * LCM(8, 6) = 8*6/GCD(8,6) = 24 bit
    * Input: 24 / 8 = 3 bytes
    * Output: 24  / 6  = 4 bytes
    * Ratio (Output/Input): 4/3

2. Base32 (Input: 8 bits, Output: 5 bits):
    * LCM(8, 5) = 8*5/GCD(8,5) = 40 bit
    * Input: 40 / 8 = 5 bytes
    * Output: 40  / 5  = 8 bytes
    * Ratio (Output/Input): 8/5

3. Base16 (Input: 8 bits, Output: 4 bits): 
    * LCM(8, 4) = 8*4/GCD(8,4) = 8 bit
    * Input: 8 / 8 = 1 byte
    * Output: 8  / 4  = 2 bytes
    * Ratio (Output/Input): 2/1

What's the point?
=================

The point is the following. What if a channel is able to transmit only just a few different symbols, like 9 or 17. That is, we have a file represented by 256 symbols alphabet (a normal 8-bit byte), we are not limited by computational power or memory constraints of both sides, but we are able to send only 7 different symbols instead of 256? Base64/32/16 are not suitable here. Then, Base7 is the only possible output format.

Another example, what if an amount of transmitted data is a concern for a channel? Base64, as it's been shown, increases the data by 33% no matter what is transmitted, always. For instance, Base94 increases output by only 22%.

It might seem that Base94 is not the limit. If the first 32 ASCII codes are control characters and there are 256 codes in total, what stops one from using an alphabet of 256 - 32 = 224 symbols? There is a reason. Not all of 224 `ASCII codes`_ have printable characters. In general, only 7 bits (0..127) are standardized and the rest (128..255) is used for the variety of locales, e.g. Koi8-R, Windows-1251, etc. That means, only 128 - 32 = 96 are available in the standardized range. In addition, the ASCII code 32 is the space character and 127 doesn't have a visible character either. Hense, 96 - 2 gives us that 94 printable characters which have the same association with their codes on all possible machines.

Solution
========

This solution is pretty simple but this simplicity also puts a significant computational constraint. The whole input file can be treated as one big number with a base 256. It might easily be areally big number required thousands of bits. Then, we just need to convert this big number to a different base. That's it. And Python3 makes it even simpler! Usually, conversions between different bases are done via an intermediate Base10. The good news is that Python3 has built-in support for big numbers calculations (it is integrated with Int), and the Int class has a method that reads any number of bytes and automatically represents them in a big Base10 number with a desired endian. So, all these complications are able to be implemented in just two lines of code, which is pretty amazing!

.. code-block:: python

    with open('input_file', 'rb') as f:
        in_data = int.from_bytes(f.read(), 'big')

Here, in_data is our big number with Base10. These are just two lines but this is the point where most computation happens and the most time is consumed. So now, convert it to any other base as it's usually done with normal small decimal numbers.

.. Links
.. _`RFC 4648`: https://tools.ietf.org/html/rfc4648
.. _`ASCII codes`: https://www.ascii-code.com/
.. _nibbles: https://en.wikipedia.org/wiki/Nibble
