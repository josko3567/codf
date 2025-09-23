# C0DF (Computer Optimized Data Format) spec.

Three parts to the file format:
  1. *C0DF Header*             - Holds information such as version, signature, offsets, etc.
  2. *Version Specific Header* - Holds information specific to the version.
  3. *Compression Header*      - Holds compression related data.
  4. *Key Header*              - Header for *Key Data*, holds values relating to all keys.
  5. *Key Data*                - Holds the names of each key, their structure, types, value data offsets into the file, flags, etc.
  6. *Value Data*              - Holds the value of the key its related to.

Note: all values are stored in little-endian.

Note: all headers are stored in the order above, missing headers don't affect the order.

# Indepth

C0DF Header
-----------

Version Mutable
: Partial, Reserved can be modified per version.

Total size
: 32 bytes

For the following versions, this header will hold these values:

### C0DF Header for C0DF Ver. 00

| Value            | Size (bytes) | Info                                                               |
|-----------------:|:------------:|:-------------------------------------------------------------------|
| Signature        | 4            | Hexadecimal value 0xc0df for validation                            |
| Version          | 2            | Version value from 0 to UINT16_MAX                                 |
| Reserved         | 2            | Future versions might use this, for now just for padding           |
| VSH Offset       | 8            | Version Specific Header file offset, 0 if none                     |
| CH Offset        | 8            | Compression Header file offset, 0 if none                          |
| KH Offset        | 8            | Key Header file offset, 0 if none                                  |
| KD Offset        | 8            | Key Data file offset, 0 if none                                    |
| VD Offset        | 8            | Value Data file offset, 0 if none                                  |
| Checksum         | 8            | Checksum used to verify the file integrity, sums all except self gi  |

Note: Checksum is calculated after compression, and checked before decompression, if the compression header exists. 

Version Specific Header
-----------------------

Version Mutable
: Yes, fully dependent on the version

Total size
: 0..n bytes

For the following versions, this header will hold these values:

### Version Specific Header for C0DF Ver. 00

*no data.*


Compression Header
------------------

Version Mutable:
: No

Special propery, Type Mutable:
: Yes, it's contents mutate depending on the type of compression.

Total size
: 0 or 8 + n byzes

For all versions, this header will hold these values:

| Value            | Size (bytes) | Info                                                          |
|-----------------:|:------------:|:--------------------------------------------------------------|
| Type             | 4            | Integer value of a supported compression type                 |
| Ver. Major       | 2            | Major version of the compression algorithim                   |
| Ver. Minor       | 2            | Minor version of the compression algorithim                   |
| Ver. Patch       | 2            | Patch version of the compression algorithim                   |
| Ver. Custom      | 6            | Checked when version is -n.-n.-n, incase of custom versioning |

For the following compression types, the header will contain additional values:

### zstd --mmp="1.5.7"

*no data.*


Key Header
----------

Version Mutable
: Partial, future additions to the header allowed but the core must stay & be filled in.

Total size
: minimaly 4 or 4 + n bytes.

For the following versions, this header will hold these values:

### Key Header for C0DF Ver. 00

| Value            | Size (bytes) | Info                                                         |
|-----------------:|:------------:|:-------------------------------------------------------------|
| No. Entries      | 8            | Number of entires in the *Key Data* part                     |


Key Data
--------

Version Mutable
: Yes, entry data can be completly changed depending on the version

Total size
: Size is equal to No. entries * the size of one entry (depends on version)

For the following versions, each entry in the key data field contains these values:

### Key Header for C0DF Ver. 00

Total size
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
