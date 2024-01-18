# Intergalactic Most Wanted List
A database for malware checksums for use with the [WallE](https://github.com/usegalaxy-eu/WallE) malware cleanup script.

This database in form of a yaml file stores malware checksums and its metadata
schema:
~~~yaml
class:
  program:
    version:
      severity: [high, medium, low]
      description: "optional info"
      checksums:
        crc32: <checksum crc32, gzip algorithm, integer representation>
        sha1: <checksum sha1, hex representation>
~~~
Contributions are always welcome. Here is a more detailed explanation how the schema works.
### Class
What purpose does it serve, e.g. mining, (port-)scanner, multi-purpose... . It can be an arbitrary yaml-compatible string.

`nmap` would **not** be a `class`, because it is a specific tool for scanning and would be a `program` under the `scanning` class.

### Program
I did not come up with a better title for now. (Refused to call it 'tool')  
Feel free to rename this.  

A `program` is every single unit of malware. In most cases it would be an executable file. But it can be a script, a binary, a package or a zip archive.  

### Version

Ideally, a version corresponds to the version of your program. In case it does not have a version of you don't know, provide an arbitrary version for example `1`, `2` so we can distinguish different checksum from the `program`.
### Checksums
Most important part first: the checksums.
The WallE script is currently using `CRC-32` as fist level quick scan method and `SHA-1` for excluding false-positives. CRC-32 is known for having hash collisions more frequently than other algorithms.
This mechanism shall combine both speed and security.
SHA-1 is pretty much self explanatory and you can use
any tool you like to calculate the hashes.  
:warning:**However this is not true for CRC-32** 
`CRC-32` has multiple different implementations, that will lead to different results. They are based on different generator polynomials, starting values and other details. You can find an overview [here](https://reveng.sourceforge.io/crc-catalogue/all.htm#crc.cat-bits.32).  
The algorithm we use is specified by [RFC in the GZIP specification](https://www.rfc-editor.org/rfc/rfc1952#page-11) and also used by `zlib` Python module.


~~~
Generator polynomial: 0x04c11db7
Initial value:      0
~~~

:warning:**The BZIP2 and IEEE standard uses a different algorithm and can not be used.**

This is the solution I currently use (mira-miracoli, 1/2023), please feel free to suggest something better:
~~~sh
gzip -1 -c /path/to/file | tail -c8 | hexdump -n4 -e '"%u"'
~~~

### Description
Plase describe what the program does. If it was described in another version of the program, you don't need to repeat that, but please point out particularities about the version you are adding. This can also include the date of first appearance.
### Severity

Why is it bad and how bad is it?
Please feel free to contribute and change :)

#### High
- criminal activity
- threatens privacy of users
- threatens integrity of data
- corrupts services

#### Medium
- abuses services without doing general harm
- binds compute resources for private profit (mining)
- violates T&S without being a high threat as described above

#### Low
- no direct harm
- not directly violating T&S but also not using the service in usual way
- creates 'unnecessary' huge load for our infrastructure
