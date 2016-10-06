# FuriousPotato
HorrorHo's Furious Potato.

### What is it?
Java [Heimdal](https://github.com/heimdal/heimdal) [ASN1](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One) template ripper. It's able to rip [asn1_template](https://github.com/heimdal/heimdal/blob/master/lib/asn1/asn1-template.h) structures from binary files and reconstruct ASN1 templates.

It's a tidied version of a private tool I created whilst reverse engineering binaries for [InflatableDonkey](https://github.com/horrorho/InflatableDonkey)'s [DER](https://en.wikipedia.org/wiki/Distinguished_Encoding_Rules#DER_encoding) [management](https://github.com/horrorho/InflatableDonkey/tree/master/src/main/java/com/github/horrorho/inflatabledonkey/data/der).

### Sample Output Snippets
Templates:
```
TEMPLATES (asn1_template):
ADDRESS : < TT      , OFFSET  , PTR      > : DESCRIPTION
10068cb0: < 00000000, 0000001c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068cbc: < 30600001, 00000000, 100688a8 > : A1_OP_TAG TAG=1 CONSTRUCTED APPLICATION 
100688a8: < 00000000, 0000001c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100688b4: < 30200010, 00000000, 10068b90 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068b90: < 00000000, 0000001c, 00000006 > : A1_OP_HEADER ELEMENTS=6 
10068b9c: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
```

Detailed Inform:
```
LOCATION > TARGET   : INFORM (experimental):
10068cbc > 100688a8 :   appl[1]
100688b4 > 10068b90 :    SEQUENCE (6)
10068c68 > 10068b24 :     INTEGER
10068c68 > 10068b24 :     INTEGER
10068bb4 > 10068d7c :     OCTET_STRING
10068bc0 > 100689b0 :     cont[0] OPTIONAL
10068854 > 10068830 :      SEQUENCE
10068a40 > 10068a4c :       SEQUENCE (2)
10068c68 > 10068b24 :        INTEGER
10068a64 > 10068d7c :        OCTET_STRING
```

Inform:
```
INFORM (experimental):
  appl[1]
   SEQUENCE (6)
    INTEGER
    INTEGER
    OCTET_STRING
    cont[0] OPTIONAL
     SEQUENCE
      SEQUENCE (2)
       INTEGER
       OCTET_STRING
```

### Build
Requires [Java 8 JRE/ JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html) and [Maven](https://maven.apache.org).

[Download](https://github.com/horrorho/FuriousPotato/archive/master.zip), extract and navigate to the FuriousPotato-master folder:

```
~/FuriousPotato-master $ mvn package
```
The executable Jar is located at /target/FuriousPotato.jar

### Usage
```
~/FuriousPotato-master/target $ java -jar FuriousPotato.jar --help
Usage: FuriousPotato FILE DELTA ADDRESS
Horrorho's Furious Potato. Heimdal ASN1 template ripper.

     --help     display this help and exit

FILE<file> input file
DELTA<hex> file position relative to executable address
ADDRESS<hex> executable address of asn_template

Example:
FuriousPotato PCS.dll -10001800 10068980

Example details:
Rips KeySet from Apple's iCloud PCS.dll (Version: 15.0.0.10.1 CRC32: E7533650)
The DLL loads at 0x10001000
KeySet asn1_template loads at 0x10068980
KeySet asn1_template position in file 0x00067180
Delta = 0x00067180 - 0x10068980 = -0x10001800
FILE=PCS.dll DELTA=-10001800 ADDRESS=10068980

Project home:
https://github.com/horrorho/FuriousPotato
```

It's a basic reversing tool and it assumes that you know what you're doing. You'll need three arguments
- FILE, our file
- DELTA, the delta in bytes that translates the asn1_template data segment location to it's corresponding file offset
- LOCATION, the data segment location of the top asn1_template

As an example, we'll take Apple's iCloud PCS.dll (Windows Version: 15.0.0.10.1 CRC32: E7533650).

My disassembler loads the [DLL](https://en.wikipedia.org/wiki/Dynamic-link_library) with a 0x10001000 base. The top asn1_template location for KeySet is 0x10068980.

The hex dump is:
```
10068980 : 00000000 20000000 01000000 02006030 00000000 888a0610 00000000 14000000
100689a0 : 01000000 10002030 00000000 50890610 00000000 08000000 01000000 00000010
100689c0 : 00000000 48880610 00000000 20000000 01000000 10002030 00000000 b88a0610
100689e0 : 00000000 0c000000 02000000 00000010 00000000 5c8c0610 00000010 04000000
```

The corresponding hex data is found in PCS.dll at 0x00067180 using a hex editor.
```
00067180 : 00000000 20000000 01000000 02006030 00000000 888a0610 00000000 14000000
000671a0 : 01000000 10002030 00000000 50890610 00000000 08000000 01000000 00000010
000671c0 : 00000000 48880610 00000000 20000000 01000000 10002030 00000000 b88a0610
000671e0 : 00000000 0c000000 02000000 00000010 00000000 5c8c0610 00000010 04000000
```

The relative difference we require is 0x00067180 - 0x10068980 = -0x10001800.
Our tool is unopinionated/ clueless in regards to binary formats and needs to be supplied with this information.

Thus, 
- FILE = PCS.dll 
- DELTA = -10001800
- ADDRESS = 10068980.

We can now run the tool
```
~/FuriousPotato-master/target $ java -jar FuriousPotato.jar PCS.dll -10001800 10068980
FILE   : /media/cain/POO/virtualbox/shared/PCS.dll
DELTA  : 0xefffe800
ADDRESS: 0x10068980

DUMP:
10068980 : 00000000 20000000 01000000 02006030 00000000 888a0610 00000000 14000000
100689a0 : 01000000 10002030 00000000 50890610 00000000 08000000 01000000 00000010
100689c0 : 00000000 48880610 00000000 20000000 01000000 10002030 00000000 b88a0610
100689e0 : 00000000 0c000000 02000000 00000010 00000000 5c8c0610 00000010 04000000

TEMPLATES (asn1_template):
ADDRESS : < TT      , OFFSET  , PTR      > : DESCRIPTION
10068980: < 00000000, 00000020, 00000001 > : A1_OP_HEADER ELEMENTS=1 
1006898c: < 30600002, 00000000, 10068a88 > : A1_OP_TAG TAG=2 CONSTRUCTED APPLICATION 
10068a88: < 00000000, 00000020, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a94: < 30200010, 00000000, 10068cf8 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068cf8: < 00000002, 00000020, 00000006 > : A1_OP_HEADER ELEMENTS=6 ELLIPSIS
10068d04: < 3000000c, 00000000, 10068878 > : A1_OP_TAG TAG=12(UTF8STRING) PRIMITIVE UNIVERSAL 
10068878: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068884: < 4000000c, 00000000, 00000000 > : A1_OP_PARSE UTF8STRING PRIMITIVE UNIVERSAL 
10068d10: < 10000000, 00000004, 10068c98 > : A1_OP_TYPE 
10068c98: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068ca4: < 30200011, 00000000, 10068b0c > : A1_OP_TAG TAG=17(SET) CONSTRUCTED UNIVERSAL 
10068b0c: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b18: < 60000000, 00000000, 10068af4 > : A1_OP_SETOF
10068af4: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b00: < 10000000, 00000000, 10068be4 > : A1_OP_TYPE 
10068be4: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068bf0: < 30200010, 00000000, 10068c74 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068c74: < 00000000, 0000000c, 00000002 > : A1_OP_HEADER ELEMENTS=2 
10068c80: < 30000004, 00000000, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068c8c: < 11000000, 00000008, 10068cb0 > : A1_OP_TYPE OPTIONAL
10068cb0: < 00000000, 0000001c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068cbc: < 30600001, 00000000, 100688a8 > : A1_OP_TAG TAG=1 CONSTRUCTED APPLICATION 
100688a8: < 00000000, 0000001c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100688b4: < 30200010, 00000000, 10068b90 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068b90: < 00000000, 0000001c, 00000006 > : A1_OP_HEADER ELEMENTS=6 
10068b9c: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068ba8: < 10000000, 00000004, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068bb4: < 30000004, 00000008, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068bc0: < 31a00000, 00000010, 100689b0 > : A1_OP_TAG TAG=0 CONSTRUCTED CONTEXT_SPECIFIC OPTIONAL
100689b0: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100689bc: < 10000000, 00000000, 10068848 > : A1_OP_TYPE 
10068848: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068854: < 30200010, 00000000, 10068830 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068830: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
1006883c: < 50000000, 00000000, 10068adc > : A1_OP_SEQOF
10068adc: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068ae8: < 10000000, 00000000, 10068a34 > : A1_OP_TYPE 
10068a34: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a40: < 30200010, 00000000, 10068a4c > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068a4c: < 00000000, 0000000c, 00000002 > : A1_OP_HEADER ELEMENTS=2 
10068a58: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068a64: < 30000004, 00000004, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068bcc: < 31a00001, 00000014, 10068a04 > : A1_OP_TAG TAG=1 CONSTRUCTED CONTEXT_SPECIFIC OPTIONAL
10068a04: < 00000000, 00000014, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a10: < 10000000, 00000000, 10068998 > : A1_OP_TYPE 
10068998: < 00000000, 00000014, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100689a4: < 30200010, 00000000, 10068950 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068950: < 00000000, 00000014, 00000003 > : A1_OP_HEADER ELEMENTS=3 
1006895c: < 10000000, 00000000, 10068d64 > : A1_OP_TYPE 
10068d64: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d70: < 30000004, 00000000, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068968: < 10000000, 00000008, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068974: < 30000004, 0000000c, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068bd8: < 31a00002, 00000018, 10068aa0 > : A1_OP_TAG TAG=2 CONSTRUCTED CONTEXT_SPECIFIC OPTIONAL
10068aa0: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068aac: < 10000000, 00000000, 100688c0 > : A1_OP_TYPE 
100688c0: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100688cc: < 30200010, 00000000, 1006880c > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
1006880c: < 00000000, 00000008, 00000002 > : A1_OP_HEADER ELEMENTS=2 
10068818: < 31a00000, 00000000, 100689b0 > : A1_OP_TAG TAG=0 CONSTRUCTED CONTEXT_SPECIFIC OPTIONAL
100689b0: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100689bc: < 10000000, 00000000, 10068848 > : A1_OP_TYPE 
10068848: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068854: < 30200010, 00000000, 10068830 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068830: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
1006883c: < 50000000, 00000000, 10068adc > : A1_OP_SEQOF
10068adc: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068ae8: < 10000000, 00000000, 10068a34 > : A1_OP_TYPE 
10068a34: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a40: < 30200010, 00000000, 10068a4c > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068a4c: < 00000000, 0000000c, 00000002 > : A1_OP_HEADER ELEMENTS=2 
10068a58: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068a64: < 30000004, 00000004, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068824: < 31a00001, 00000004, 10068a04 > : A1_OP_TAG TAG=1 CONSTRUCTED CONTEXT_SPECIFIC OPTIONAL
10068a04: < 00000000, 00000014, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a10: < 10000000, 00000000, 10068998 > : A1_OP_TYPE 
10068998: < 00000000, 00000014, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100689a4: < 30200010, 00000000, 10068950 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068950: < 00000000, 00000014, 00000003 > : A1_OP_HEADER ELEMENTS=3 
1006895c: < 10000000, 00000000, 10068d64 > : A1_OP_TYPE 
10068d64: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d70: < 30000004, 00000000, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068968: < 10000000, 00000008, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068974: < 30000004, 0000000c, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068d1c: < 30200011, 0000000c, 10068cc8 > : A1_OP_TAG TAG=17(SET) CONSTRUCTED UNIVERSAL 
10068cc8: < 00000000, 00000020, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068cd4: < 60000000, 00000000, 100688d8 > : A1_OP_SETOF
100688d8: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
100688e4: < 10000000, 00000000, 100687f4 > : A1_OP_TYPE 
100687f4: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068800: < 30200010, 00000000, 100689e0 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
100689e0: < 00000000, 0000000c, 00000002 > : A1_OP_HEADER ELEMENTS=2 
100689ec: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
100689f8: < 10000000, 00000004, 10068d64 > : A1_OP_TYPE 
10068d64: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d70: < 30000004, 00000000, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068d28: < 31000004, 00000014, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL OPTIONAL
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 
10068d34: < 11000000, 00000018, 10068c5c > : A1_OP_TYPE OPTIONAL
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068d40: < 11000000, 0000001c, 10068848 > : A1_OP_TYPE OPTIONAL
10068848: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068854: < 30200010, 00000000, 10068830 > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068830: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
1006883c: < 50000000, 00000000, 10068adc > : A1_OP_SEQOF
10068adc: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068ae8: < 10000000, 00000000, 10068a34 > : A1_OP_TYPE 
10068a34: < 00000000, 0000000c, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068a40: < 30200010, 00000000, 10068a4c > : A1_OP_TAG TAG=16(SEQUENCE) CONSTRUCTED UNIVERSAL 
10068a4c: < 00000000, 0000000c, 00000002 > : A1_OP_HEADER ELEMENTS=2 
10068a58: < 10000000, 00000000, 10068c5c > : A1_OP_TYPE 
10068c5c: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068c68: < 30000002, 00000000, 10068b24 > : A1_OP_TAG TAG=2(INTEGER) PRIMITIVE UNIVERSAL 
10068b24: < 00000000, 00000004, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068b30: < 40000003, 00000000, 00000000 > : A1_OP_PARSE BIT_STRING PRIMITIVE UNIVERSAL 
10068a64: < 30000004, 00000004, 10068d7c > : A1_OP_TAG TAG=4(OCTET_STRING) PRIMITIVE UNIVERSAL 
10068d7c: < 00000000, 00000008, 00000001 > : A1_OP_HEADER ELEMENTS=1 
10068d88: < 40000005, 00000000, 00000000 > : A1_OP_PARSE NULL PRIMITIVE UNIVERSAL 

LOCATION > TARGET   : INFORM (experimental):
1006898c > 10068a88 :   appl[2]
10068a94 > 10068cf8 :    SEQUENCE (6)
10068d04 > 10068878 :     UTF8STRING
10068ca4 > 10068b0c :     SET
10068bf0 > 10068c74 :      SEQUENCE (2)
10068c80 > 10068d7c :       OCTET_STRING
10068cbc > 100688a8 :       appl[1] OPTIONAL
100688b4 > 10068b90 :        SEQUENCE (6)
10068c68 > 10068b24 :         INTEGER
10068c68 > 10068b24 :         INTEGER
10068bb4 > 10068d7c :         OCTET_STRING
10068bc0 > 100689b0 :         cont[0] OPTIONAL
10068854 > 10068830 :          SEQUENCE
10068a40 > 10068a4c :           SEQUENCE (2)
10068c68 > 10068b24 :            INTEGER
10068a64 > 10068d7c :            OCTET_STRING
10068bcc > 10068a04 :         cont[1] OPTIONAL
100689a4 > 10068950 :          SEQUENCE (3)
10068d70 > 10068d7c :           OCTET_STRING
10068c68 > 10068b24 :           INTEGER
10068974 > 10068d7c :           OCTET_STRING
10068bd8 > 10068aa0 :         cont[2] OPTIONAL
100688cc > 1006880c :          SEQUENCE (2)
10068818 > 100689b0 :           cont[0] OPTIONAL
10068854 > 10068830 :            SEQUENCE
10068a40 > 10068a4c :             SEQUENCE (2)
10068c68 > 10068b24 :              INTEGER
10068a64 > 10068d7c :              OCTET_STRING
10068824 > 10068a04 :           cont[1] OPTIONAL
100689a4 > 10068950 :            SEQUENCE (3)
10068d70 > 10068d7c :             OCTET_STRING
10068c68 > 10068b24 :             INTEGER
10068974 > 10068d7c :             OCTET_STRING
10068d1c > 10068cc8 :     SET
10068800 > 100689e0 :      SEQUENCE (2)
10068c68 > 10068b24 :       INTEGER
10068d70 > 10068d7c :       OCTET_STRING
10068d28 > 10068d7c :     OCTET_STRING OPTIONAL
10068c68 > 10068b24 :     INTEGER OPTIONAL
10068854 > 10068830 :     SEQUENCE OPTIONAL
10068a40 > 10068a4c :      SEQUENCE (2)
10068c68 > 10068b24 :       INTEGER
10068a64 > 10068d7c :       OCTET_STRING
10068cf8 > N/A      :    ...

INFORM (experimental):
  appl[2]
   SEQUENCE (6)
    UTF8STRING
    SET
     SEQUENCE (2)
      OCTET_STRING
      appl[1] OPTIONAL
       SEQUENCE (6)
        INTEGER
        INTEGER
        OCTET_STRING
        cont[0] OPTIONAL
         SEQUENCE
          SEQUENCE (2)
           INTEGER
           OCTET_STRING
        cont[1] OPTIONAL
         SEQUENCE (3)
          OCTET_STRING
          INTEGER
          OCTET_STRING
        cont[2] OPTIONAL
         SEQUENCE (2)
          cont[0] OPTIONAL
           SEQUENCE
            SEQUENCE (2)
             INTEGER
             OCTET_STRING
          cont[1] OPTIONAL
           SEQUENCE (3)
            OCTET_STRING
            INTEGER
            OCTET_STRING
    SET
     SEQUENCE (2)
      INTEGER
      OCTET_STRING
    OCTET_STRING OPTIONAL
    INTEGER OPTIONAL
    SEQUENCE OPTIONAL
     SEQUENCE (2)
      INTEGER
      OCTET_STRING
   ...

```

### How do I reverse binaries and find/ identify templates?
Sadly this is beyond the scope of our discussion.












