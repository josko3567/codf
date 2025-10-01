# CODF (Computer Optimized Data Format) spec.

Three parts to the file format:
  1. *Main CODF Header*        - Holds information such as version, signature, offsets, etc.
  2. *Version Specific Header* - Holds information specific to the version.
  3. *Checksum Header*         - Holds information on the type of checksum and the checksum itself.
  4. *Compression Header*      - Holds compression related data.
  5. *Key Header*              - Header for *Key Data*, holds values relating to all keys.
  6. *Key Data*                - Holds the names of each key, their structure, types, value data offsets into the file, flags, etc.
  7. *Value Data*              - Holds the value of the key its related to.

Note: all values are stored in little-endian.

Note: all headers are stored in the order above, missing headers don't affect the order.

# Indepth

*Main CODF Header*
-------------

__Value Mutable__
: No

__Version Mutable__
: No, unmodifiable for compatibility with future versions.

__Total Size__
: 64 bytes

*Main CODF Header* is the main header of the file, once I finish its conception it will never change for compatibility
with newer and older versions of *CODF*.

It holds the main *Signature* `0xc0dfc0df` to verifiy we are handling a *CODF* file. 
Since both `0xc0` and `0xdf` are outside of the 0 - 127 range of ASCII it's pretty good verification that we are indeed parsing a CODF file (other than by the extension `.codf`).

*Version Major, Minor & Patch* will entail the version of *CODF*. Starting from **1.0.0**, its inceptual version, every update that modifies the data structure of the file and/or changes the internal functionality of the *serializer / deserializer* will be a new version. 

Simple things as adding or even removing sections of any header (except the main *CODF Header*), creating new or removing old headers (again, except the main *CODF Header*) & changing how *Key Data* and *Value Data* is stored in any shape or form requires a new version. Aswell as the internal functionality of *CODF*, any change of the internal functionality requires a new version.

For example if we were to add a new value to the key header named *Minimum Keys*, this will require a new version. Removing the *Compression Header* will require a new version. Even moving a value around like changing how *Key Data* is stored will require a new version. If we change the way we parse a integer or float, that requires a new version and so on.

2 bytes are *Reserved* for padding.

Offsets for all the other headers starting from the beggining of the file aka. `fseek(stream, offset, SEEK_SET)`. Important to know that if a offset is equal to 0 that means it doesn't exist within the file.

For what offsets can be 0, the short version is you can have all offsets be equal to 0.
As for the long version not really, if for example you have *Key Data* offset != 0 of course you need a *Key Header* offset that is != 0.
Same goes for *Value Data* offset, it's gonna need a *Key Data* offset pair. If your data is compressed, of course you will need a *Compression Header* offset and so on. But generaly, if you literaly store nothing then yes all offsets can be 0.

For the following versions, this header will hold these values in the order from top to bottom:

### *Main CODF Header* for *CODF* Version:
### 1.0.0

| Value            | Size (bytes) | Info                                                               |
|-----------------:|:------------:|:-------------------------------------------------------------------|
| Signature        | 8            | Hexadecimal value `0xc0dfc0df` for file type validation            |
| Version Minor    | 2            | Version value from 0 to `INT16_MAX`                                |
| Version Patch    | 2            | Version value from 0 to `INT16_MAX`                                |
| Version Major    | 2            | Version value from 0 to `INT16_MAX`                                |
| Reserved         | 2            | Padding                                                            |
| VSH Offset       | 8            | *Version Specific Header* file offset, 0 if none                   |
| CS Offset        | 8            | *Checksum Header* file offset, 0 if none                           |
| CH Offset        | 8            | *Compression Header* file offset, 0 if none                        |
| KH Offset        | 8            | *Key Header* file offset, 0 if none                                |
| KD Offset        | 8            | *Key Data* file offset, 0 if none                                  |
| VD Offset        | 8            | *Value Data* file offset, 0 if none                                |

Note: Future offsets will be either stored in the *Version Specific Header* or in a header pointed to by a value in the *Version Specific Header*.

*Version Specific Header*
-------------------------

__Value Mutable__
: No

__Version Mutable__
: Yes, fully dependent on the version

__Total Size__
: 0..n bytes

The *Version Specific Header* is used to store any data for future versions. It's basically the modifiable part of the *Main CODF Header*.

What data it stores is really reliant on the version, but it can store anything. But the main purpose I created it for is the storage of offsets for new headers (in the future).

For the following versions, this header will hold these values:

### *Version Specific Header* for *CODF* Version:
### 1.0.0

*no values.*


*Compression Header*
--------------------

__Value Mutable__
: Yes, it's contents mutate depending on the type of compression aka. depending on the value *Type*.

__Version Mutable__
: Partial, the extra contents the header contains depending on the value *Type* can be different per version & more values to *Type* can be added / moved around aswell.

__Total Size__
: 0 or 16 + n bytes

For all versions, this header will hold these values:

| Value            | Size (bytes) | Info                                                          |
|-----------------:|:------------:|:--------------------------------------------------------------|
| Type             | 4            | Integer value of a supported compression type                 |
| Ver. Major       | 2            | Major version of the compression algorithim                   |
| Ver. Minor       | 2            | Minor version of the compression algorithim                   |
| Ver. Patch       | 2            | Patch version of the compression algorithim                   |
| Ver. Custom      | 6            | Checked when version is -n.-n.-n, incase of custom versioning |

For the following compression types, the header will contain additional values:

### `zstd` Version (using Major.Minor.Patch): 
### 1.5.7

*no values.*

Key Header
----------

__Value Mutable__
: No

__Version Mutable__
: Partial, future additions to the header allowed but the core must stay & be filled in.

__Total Size__
: Minimum 8 or 8 + n bytes.

For the following versions, this header will hold these values:

### Key Header for CODF Ver. 00

| Value            | Size (bytes) | Info                                                         |
|-----------------:|:------------:|:-------------------------------------------------------------|
| No. Entries      | 8            | Number of entires in the *Key Data* part                     |


Key Data
--------

__Value Mutable__
: No

__Version Mutable__
: Yes, key data can be completly changed depending on the version

__Total Size__
: Size is equal to No. entries * the size of one entry (depends on version)

For the following versions, each entry in the key data field contains these values:

### Key Header for CODF Ver. 00

__Total Size__
: 24 bytes + Key Name Size + Pad

| Value            | Size (bytes) | Info                                                         |
|-----------------:|:------------:|:-------------------------------------------------------------|
| Type             | 1            | Type of the related value to this key                        |
| Flags            | 1            | Custom flags, stored with each individual bit in the byte    |
| Depth            | 2            | Current depth of the key, indicating keys that contain keys  |
| Key Name Length  | 2            | Length of the key name string                                |
| Reserved         | 2            | Padding                                                      |
| Value Offset     | 8            | File offset where the value data starts, starting from 0     |
| Value Length     | 8            | Size of the value in bytes                                   |
| Key Name         | KN Len. + Pad| Name of the key                                              |

Note: Key Name's size in memory is KN Len. rounded to the nearest multiple of 4 (hence the + Pad)
