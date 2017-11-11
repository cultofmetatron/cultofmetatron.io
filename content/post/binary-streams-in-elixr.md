---
Categories:
- Development
- elixir
Description: "binary stream handling in elixir"
Tags:
- Development
- elixir
date: 2016-03-02T00:16:51-05:00
menu: blog
title: binary streams in elixr
---

Everything we touch on a computer is just 1's and 0's. Yet, its easy to forget that with all the libraries we have to abstract away the innate complexity of the myriad of protocols that form the backbone of the internet.
<!--more--> 
Elixir itself has really nice built in primatives for dealing with binary streams. Before I go into that, lets cover some background.

### Hexadecimal Notation

For thousands of years, humans have done just fine with a base 10 number system. Why use hexadecimal ( a base 16 representation) for computers? There are some very practical reasons.

A bit is a single 1 or 0 value. It can represent 2 values.  If we want to have 2 bits, we can represent up 4 values

* 00
* 01
* 10
* 11

if we have 3 values, this effectivly doubles the total set of possible combinations. More generally, the size N of the bit stream yields 2^N diffrent representable values. A nibble, consisting of 4 bits can represent 2^4 or 16 possible values. There are 8 bits or 2 nibbles in a byte.

since we have more symbols to work with, the decimal set of numbers

0 1 2 3 4 5 6 6 7 8 9 10 11 12 13 14 15

becomes

0 1 2 3 4 4 5 6 7 8 9 a b c d e f

since nibbles and bytes are a standard grouping of bits, by using hexadecimal, we can represent bytestreams directly in hexadecimal.

* 0  0000  0x0
* 1  0001  0x1
* 2  0010  0x2
* 3  0011  0x3
* 4  0100  0x4
* 5  0101  0x5
* 6  0110  0x6
* 7  0111  0x7
* 8  1000  0x8
* 9  1001  0x9
* 10 1010  0xA
* 11 1011  0xB
* 12 1100  0xC
* 13 1101  0xD
* 14 1110  0xE
* 15 1111  0xF

so the stream `0001 1011 1111 0100` can be more sucictly described as `0x1BF4`. Put another way, two hex numbers form a 1:1 cannonical representation for a byte.

### Elixir bytestreams

Elixir is really flexible and clean when it comes to bytestream literals. 

lets say we cwant to encode `0001 1011 1111 0100` as a direct value in elixir, well we already know that `0x1BF4` is its direct value in hex.

Naively, you could try 

```
iex> <<0x1bf4>>
<<244>>
```

by default, elixir outputs thhe stream contents in decimal format. 244 translates to F4. The second set of arguments. By default, each entry is assumed to be 1 byte. We can modify this with the `::size()` option

```
iex> <<0x1bf4::size(16)>>
<<27, 244>>
```

What this means is that we are goingto take 16 bits or 2 bytes. alternativly, we could specify the two halves as two seperate entries.

```
iex> <<0x1b, 0xbf4>>
<<27, 244>>
```

### Endianess

lets take the number 1025. The digit that is most significant is the one that contributes the most to the value of the number. in this case, the 1 which is farthest on the left is the most significant since 2025 is more significantly diffrent from 1026. We are so used to this in decimal tat few ever really think about it being any other way. On computers with binary, this is somethign worth considering. depending on the context, 0001 could mean 0x1 or it could mean 0x8 depending on which side you want to be the most significant. This property is known as *endianess*. 

When the most significan bit is on the far left, we call this *Big Endian*

when its on the far right, we call it *Little Endian*.

We can specify the endianess when we create the stream.

```

iex> <<0x1bf4::little-size(16)>>
<<244, 27>>
iex> <<0x1bf4::little-size(32)>>
<<244, 27, 0, 0>>
iex> <<0x1bf4::big-size(32)>>
<<0, 0, 27, 244>>

```

### Concatting streams

concatting streams is easy with the `<>` operator

```
iex> <<0x1b>> <> <<0xbf4>>
<<27, 244>>
```

Coming from a node background, this is way nicer than using Buffer. Elixir lets me treat my binary streams as immutable values.

### Extracting Bytes

Lets say you have a Binary stream that you've gotten over a udp or tcp socket and you want to pull values out of it. knwoing where to address is part of the protocol you are implementing. 

In bit torrent, an announce message as recieved over a UDP socket is specified as the following.

Borrowed from [this lovely tutorial](http://allenkim67.github.io/bittorrent/2016/05/04/how-to-make-your-own-bittorrent-client.html?utm_source=nodeweekly&utm_medium=email#connect-messaging)

```

Offset  Size            Name            Value
0       32-bit integer  action          0 // connect
4       32-bit integer  transaction_id
8       64-bit integer  connection_id
16

```

if we had a 16 byte binary stream, we know from the spec where we should start slicing the various sections of the stream.

Two built in elixir operators provide a great service.

* [byte_size(stream)](http://elixir-lang.org/docs/stable/elixir/Kernel.html#byte_size/1) : returns the number of bytes
* [binary_part(stream, start, length)](http://elixir-lang.org/docs/stable/elixir/Kernel.html#binary_part/3) : returns the subsection of the stream.

Given the spec for the proticol, elixir makes parsing the raw output into somethign more managable within my code easy. 

```elixir
  def parse_connect_response(resp) do
    case byte_size(resp) do
      16 ->
        {:ok, %{
          action: get_action(binary_part(resp, 0, 4)),
          transaction_id: binary_part(resp, 4, 4),
          connection_id: binary_part(resp, 8, 8)
        }}
      _ -> {:error, :bad_response}
    end
  end

```

