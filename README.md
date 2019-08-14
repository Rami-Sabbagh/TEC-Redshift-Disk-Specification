# TEC Redshift Disk Specification (Revision 001)
A specification of the TEC Redshift exported images format in the [EXAPUNKS](http://www.zachtronics.com/exapunks) game by [Zachtronics](https://www.zachtronics.com/).

# Values structures:

## Bool
- 1-byte, 0 for _false_, 1 for _true_

## Byte
- 1-byte, it's a byte ! :P

## Int
- 4-bytes, little-endian format

## String
- The string's length (Int)
- The actual string data, an sequence of characters

# The disk raw data structure:
The disk raw data has the following structure:
- Compressed solution data's length (Int)
- Compressed solution data's checksum (Int)
- Compressed solution data (sequence of bytes)
- Garbage data (unknown length)

## Dumping the disk raw data:
The disk raw data, is stored in the image pixels color values, inorder to dump it, follow this protocol:

### 1. Read the bitstream:

1. Iterate over (map) the image pixels, from top-left, into the bottom-right, line by line.
2. Read the first bit of each of RGB channel values (ignore the alpha ones), and store it into a _"bitstream array"_

### 2. Convert the bit stream into bytes:

1. Put every 8 bits into a byte, the first bit in the stream is the least signicant bit of a byte, the bitstream could be visualized as
```
| 1 2 4 8 16 32 64 128 | 1 2 4 8 16 32 64 128 | ...
  ^                    --->
First bit         Stream flow
```
2. Insert each resulting byte into a bytes array, or you could write them into a file directly, as you wish in your implementation.

## Compressed solution data's length (Int):
An int, which is the length (in bytes) of the compressed solution data, you'll have to read it so you know how to extract the compressed solution data from the raw disk data and not include the garbage.

## Compressed solution data's checksum (Int):
An int, which is a checksum of the compressed solution data, it's optional to verify it, but better do that.

The checksum algorithm used is [Fletcher-16 checksum](https://en.wikipedia.org/wiki/Fletcher%27s_checksum) (Note that the checkbytes are not used).

## Compressed solution data (sequence of bytes):
It's the game solution data, but **zlib** compressed.

## The garbage data (unkown length):
It's no useful data, just ignore it and throw it in garbage, it's there due the nature of storing the data in the image.

# Solution data:
The game solution data has the following structure:
- unknown_01 (Int)
- level_id (String)
- solution_name (String)
- unknown_02 (Int)
- solution_length (Int)
- unknown_03 (Int)
- exa_count (Int)
- exa_list

## unknown_01, unknown_02, unknown_03
They are really unknown.

> If someone find a pattern and discoveres what they are, please create a pull request.

## level_id (String)
It's the internal ID of the game's level, which is always `PB039` for the TEC Redshift solutions.

## solution_name (String)
The name of the solution, in other words, the name of the game.

## solution length (Int)
The sum of the functional instructions count for every exa

> Functional instructions are all instructions except DATA and NOTE, empty lines and ; comments are also ignored.

## exa_count (Int)
The count of exas initiated in this solution, can't be bigger than 36

> Note that in-game execution could have a higher number of total EXAs !

## exa_list
It's a sequence of EXA data structures (check below), without any separator between them.

# EXA Data structure:
Each EXA Data has the following structure:
- unknown_04 (Byte)
- exa_name (String)
- exa_code (String)
- exa_code_view_mode (Byte)
- exa_m_register_mode (Byte)
- exa_sprite (sequence of Bools)

## unknown_04 (Int)
It's really unkown, it has been found with value (10) in the all observed TEC games.
> If someone find a pattern and discoveres what it is, please create a pull request.

## exa_name (String)
The name of the EXA, it's can't be longer than 2 character.

## exa_code (String)
The code of the EXA, as written in the editor, contains new line characters and such.

## exa_code_view_mode (Int)
The mode of the EXA code windows, has the following possible values:
- [0] Default maximized view.
- [1] Minimized, no visible code or registers.
- [2] Follow current instruction.

## exa_m_register_mode (Int)
The initial mode of the M register, has the following possible values:
- [0] Global mode (Default).
- [1] Local mode.
> The M register mode could be toggled during the EXA execution using the `MODE` instruction.

## exa_sprite (sequence of Bools)
A sequence of 100 Bools, representing the sprite pixel data from top-left into the top-right, line by line.

Each pixel has the following possible values:

- [True] White.
- [False] Black/Transparent (Default).

# Outro
That's what we've discoverd about the TEC disks format so far, use it for good please ;)

**Happy hacking !**