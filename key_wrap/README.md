# Key Wrap Tool
---
## Introduction:
---

The Key wrapping tool uses AES key wrap algorithm (RFC3394) is utilized to wrap
the secrets with OTFAD key. The secret contains the Image encryption key,
Counter, start and  end address of the image being encrypted and CRC32 of the
previous information. The IV used is constant ```0xa6a6a6a6_a6a6a6a6``` as  per
RFC3394. As OTFAD engine implements 4 context, there could exist 4 different
Image encryption keys and other input data.

## Description:
---

Key blob processing is a mechanism where a preloaded data structure, wrapped
using the RFC3394 standard, is fetched from the external flash memory after a
system reset event , unwrapped by the OTFAD hardware and automatically loaded
into the four memory context programming model registers. In this manner, the
sensitive context data, for example, the 128-bit key and 64 bits of the counter,
is unwrapped and handled entirely within the OTFAD.

The key wrapping tool uses OTFAD key, IEK and counter as inputs along with the
start and end address of the Boot image going to be encrypted. The output is 48 
bytes of keyblob generated by the tool.

### Input:
---
The key_wrap tool takes 40 byte Plaintext array, for each context, constructed 
using the OTFAD key, IEK, Counter, Start and End address as follows:

```text
+------------------------------+
|        128 bit IEK           |
|------------------------------|
|        64 bit Counter        |
|------------------------------|
|        32 bit Start Addr.    |
|------------------------------|
|        32 bit End Addr.      |
|------------------------------|
|        32 bit filler         |
|------------------------------|
| 32 bit CRC32 calculated over |
| IEK, Counter, Start Address  |
| and End Address              |
+------------------------------+
```

### Output:
---
The key_wrap tool generates 48 byte wrapped text of input plaintext called a key
blob. As there are 4 contexts, each with its own IEK, counter, start address and
end address, there are 4 key blobs generated which are then  programmed at 4
locations in the boot image as follows:

```text
+------------------------------+   <-- QSPI_BASE_ADDR + 0x0
|       KeyBlob Context 0      |
+------------------------------+   <-- QSPI_BASE_ADDR + 0x40
|       KeyBlob Context 1      |
+------------------------------+   <-- QSPI_BASE_ADDR + 0x80
|       KeyBlob Context 2      |
+------------------------------+   <-- QSPI_BASE_ADDR + 0xC0
|       KeyBlob Context 3      |
+------------------------------+
```

The output key blobs are padded as they should be aligned to the context size of
0x40 (64 bytes).

## Build:
---
```make```


## Build with DEBUG enabled:
---
```make DEBUG=1```

## Clean:
---
```make clean```

## Usage:
---
```text
    ./key_wrap (Sample test values used. Output is stdout.)
    ./key_wrap -i <otfad-key> -k <enc-key> -c <counter> -s <start-address> -e <end-address> -v <is-valid> -o <output>
Options:
    -i|--otfad-key  -->  Input OTFAD key (128-bit)
    -k|--enc-key  -->  Input Image Encryption Key (128-bit)
    -c|--counter  -->  Input counter (64-bit)
    -s|--start-address  -->  Start address (32-bit)
    -e|--end-address  -->  End address (32-bit)
    -v|--is-valid  -->  Valid bit
    -o|--output  -->  Output File
    -h|--help  -->  This text
```

## Examples:
---
```text
./key_wrap --otfad-key otfad_key --enc-key key --counter ctr --start-address 0xC0001000 --end-address 0xC0008000 --is-valid --output blob0

./key_wrap -i otfad_key -k key -c ctr -s 0xC0001000 -e 0xC0008000 -v -o blob0
```