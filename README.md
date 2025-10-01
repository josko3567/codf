# Computer Optimized Data Format

Specifications & a library implementation (in *Rust*) of my data format with the acronym *CODF*, *Computer Optimized Data Format*.

Prolog
------

*CODF* is a [data archive format](https://en.wikipedia.org/wiki/Archive_file) like *TAR*, *VPK* (*Valve PacK*), etc. that can store files and their file structure with the ability of having additional data like integers, strings, arrays, tables, etc. stored in a fashion very similar to a [markup language](https://en.wikipedia.org/wiki/Markup_language) like a *JSON* or a *TOML* file.

*CODF* has the flexibility of being either a *markup language*, *data archive format* or both.
However, *CODF* having the ability of being a markup language doesn't mean it can be edited like one in a basic text editor. Due to the *CO* part of *CODF*, data is stored in a binary format and not a textual format.

*CODF* will have a special editor with the ability to add, remove and modify any entry in a *CODF* file named *CODFedit*.

Along side with being able to store data, *CODF* has the ability to compress said data with a compression algorithm. Currently the only supported (well planned to be supported) compression algoritihm is [zstd](https://en.wikipedia.org/wiki/Zstd).

Specifications
--------------

For specs, head over to [specs.md](./misc/specs/specs.md) in ./misc/specs/.
