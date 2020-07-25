# hash-wasm

[![codecov](https://codecov.io/gh/Daninet/hash-wasm/branch/master/graph/badge.svg)](https://codecov.io/gh/Daninet/hash-wasm)

Hash-WASM is a fast and portable hash function library.

It is using WebAssembly to calculate the hash faster than other JavaScript-based implementations.


Supported hash functions
=======

- BLAKE2b
- CRC32
- MD4, MD5
- RIPEMD-160
- SHA-1
- SHA-2: SHA-224, SHA-256, SHA-384, SHA-512
- SHA-3: SHA3-224, SHA3-256, SHA3-384, SHA3-512
- Keccak: Keccak-224, Keccak-256, Keccak-384, Keccak-512
- xxHash: xxHash32, xxHash64

HMAC and PBKDF2 is also supported with all hash algorithms

Features
=======

- A lot faster than JS implementations (see [benchmarks](#benchmark) below)
- Compiled from heavily optimized algorithms written in C
- Supports all modern browsers and Node.js
- Supports large data streams
- Supports UTF-8 strings and typed arrays
- Supports chunked input streams
- Supports HMAC for all algorithms
- Works without Webpack or other bundlers
- WASM modules are bundled as base64 strings (no problems with linking)
- Supports tree shaking (it only bundles the hash algorithms you need)
- Includes TypeScript type definitions
- Works in Web Workers
- Zero dependencies
- Supports concurrent hash calculations with multiple states
- Easy to use Promise-based async API


Installation
=======
```
npm i hash-wasm
```

or it can be inserted directly into HTML (via [jsDelivr](https://www.jsdelivr.com/package/npm/hash-wasm))

```html
<script src="https://cdn.jsdelivr.net/npm/hash-wasm"></script>
<!-- defines the global `hashwasm` variable -->
```

Examples
=======

### React.js demo app

[Hash calculator](https://3w4be.csb.app/) - [React.js source code](https://codesandbox.io/s/hash-wasm-3w4be?file=/src/App.tsx)

### Usage with the shorthand form

It is the easiest and the fastest way to calculate hashes. Use it when the input buffer is already in the memory.

```javascript
import { md5, sha1, sha512, sha3 } from 'hash-wasm';

async function run() {
  console.log('MD5:', await md5('demo'));

  const int8Buffer = new Uint8Array([0, 1, 2, 3]);
  console.log('SHA1:', await sha1(int8Buffer));
  console.log('SHA512:', await sha512(int8Buffer));

  const int32Buffer = new Uint32Array([1056, 641]);
  console.log('SHA3-256:', await sha3(int32Buffer, 256));
}

run();
```

*\* See [API reference](#api)*

### Advanced usage with chunked input

createXXXX() functions create new WASM instances with separate states, which can be used to calculate multiple hashes paralelly. They are slower compared to shorthand functions like md5(), which reuse the same WASM instance and state to do multiple calculations. For this reason, the shorthand form is always preferred when the data is already in the memory.

For the best performance, avoid calling createXXXX() functions in loops. When calculating multiple hashes sequentially, the init() function can be used to reset the internal state between runs. It is faster than creating new instances with createXXXX().

```javascript
import { createSHA1 } from 'hash-wasm';

async function run() {
  const sha1 = await createSHA1();
  sha1.init();

  while (hasMoreData()) {
    const chunk = readChunk();
    sha1.update(chunk);
  }

  const hash = sha1.digest();
  console.log('SHA1:', hash);
}

run();
```

*\* See [API reference](#api)*

### Calculating HMAC

All supported hash functions can be used to calculate HMAC. For the best performance, avoid calling createXXXX() in loops (see `Advanced usage with chunked input` section above)

```javascript
import { createHMAC, createSHA3 } from 'hash-wasm';

async function run() {
  const hashFunc = createSHA3(224); // SHA3-224
  const hmac = await createHMAC(hashFunc, 'key');

  const fruits = ['apple', 'raspberry', 'watermelon'];
  console.log('Input:', fruits);

  const codes = fruits.map(data => {
    hmac.init();
    hmac.update(data);
    return hmac.digest();
  });

  console.log('HMAC:', codes);
}

run();
```

*\* See [API reference](#api)*

### Calculating PBKDF2

All supported hash functions can be used to calculate PBKDF2. For the best performance, avoid calling createXXXX() in loops (see `Advanced usage with chunked input` section above)

```javascript
import { pbkdf2, createSHA1 } from 'hash-wasm';

async function run() {
  const iterations = 1000;
  const keyLen = 32;
  const key = await pbkdf2('password', 'salt', iterations, keyLen, createSHA1());

  console.log('Derived key:', key);
}

run();
```

*\* See [API reference](#api)*

Browser support
=====

Chrome | Safari | Firefox | Edge | IE            | Node.js
-------|--------|---------|------|---------------|--------
57+    | 11+    | 53+     | 16+  | Not supported | 8+


Benchmark
=====

You can make your own measurements here: [link](https://csb-9b6mf.daninet.now.sh/)

The source code for the benchmark can be found [here](https://codesandbox.io/s/hash-wasm-benchmark-9b6mf)

Two scenarios were measured:
- throughput with the short form (input size = 32 bytes)
- throughput with the short form (input size = 1MB)

Results:

MD5                      | throughput (32 bytes) | throughput (1MB)
-------------------------|-----------------------|-----------------
**hash-wasm**            | **27.60 MB/s**        | **609.20 MB/s**
md5 (npm library)        | 6.89 MB/s             | 11.10 MB/s
node-forge (npm library) | 6.78 MB/s             | 10.59 MB/s

#

SHA1                     | throughput (32 bytes) | throughput (1MB)
-------------------------|-----------------------|-----------------
**hash-wasm**            | **22.38 MB/s**        | **625.53 MB/s**
jsSHA (npm library)      | 4.61 MB/s             | 36.09 MB/s
crypto-js (npm library)  | 5.28 MB/s             | 14.18 MB/s
sha1 (npm library)       | 6.48 MB/s             | 11.91 MB/s
node-forge (npm library) | 6.09 MB/s             | 10.98 MB/s

#

SHA256                    | throughput (32 bytes) | throughput (1MB)
--------------------------|-----------------------|-----------------
**hash-wasm**             | **20.73 MB/s**         | **251.87 MB/s**
sha256-wasm (npm library) | 4.91 MB/s             | 175.70 MB/s
jsSHA (npm library)       | 4.24 MB/s             | 30.75 MB/s
crypto-js (npm library)   | 5.17 MB/s             | 14.11 MB/s
node-forge (npm library)  | 4.36 MB/s             | 10.28 MB/s

#

SHA512                   | throughput (32 bytes) | throughput (1MB)
-------------------------|-----------------------|-----------------
**hash-wasm**            | **15.74 MB/s**        | **372.07 MB/s**
jsSHA (npm library)      | 1.92 MB/s             | 11.61 MB/s
node-forge (npm library) | 1.94 MB/s             | 9.43 MB/s
crypto-js (npm library)  | 1.25 MB/s             | 5.74 MB/s

#

SHA3-512            | throughput (32 bytes) | throughput (1MB)
--------------------|-----------------------|-----------------
**hash-wasm**       | **14.96 MB/s**        | **175.76 MB/s**
sha3 (npm library)  | 0.87 MB/s             | 5.17 MB/s
jsSHA (npm library) | 0.78 MB/s             | 1.84 MB/s

#

XXHash64                  | throughput (32 bytes) | throughput (1MB)
--------------------------|-----------------------|------------------
**hash-wasm**             | **24.70 MB/s**        | **11882.99 MB/s**
xxhash-wasm (npm library) | 0.08 MB/s             | 47.30 MB/s
xxhashjs (npm library)    | 0.36 MB/s             | 17.74 MB/s

#

PBKDF2-SHA512 - 1000 iterations  | operations per second (16 bytes)
---------------------------------|----------------------
**hash-wasm**                    | **204 ops**         
pbkdf2 (npm library)             | 51 ops              
crypto-js (npm library)          | 6 ops               

\* These measurements were made with `Chrome v83` on a Kaby Lake desktop CPU.

API
=====

```ts
type IDataType = string | Buffer | Uint8Array | Uint16Array | Uint32Array;

// all functions return hash in hex format
blake2b(data: IDataType, bits?: number, key?: IDataType): Promise<string> // default is 512 bits
crc32(data: IDataType): Promise<string>
keccak(data: IDataType, bits?: 224 | 256 | 384 | 512): Promise<string> // default is 512 bits
md4(data: IDataType): Promise<string>
md5(data: IDataType): Promise<string>
ripemd160(data: IDataType): Promise<string>
sha1(data: IDataType): Promise<string>
sha224(data: IDataType): Promise<string>
sha256(data: IDataType): Promise<string>
sha3(data: IDataType, bits?: 224 | 256 | 384 | 512): Promise<string> // default is 512 bits
sha384(data: IDataType): Promise<string>
sha512(data: IDataType): Promise<string>
xxhash32(data: IDataType, seed?: number): Promise<string>
xxhash64(data: IDataType, seedLow?: number, seedHigh?: number): Promise<string>

createBLAKE2b(bits?: number, key?: IDataType): Promise<IHasher> // default is 512 bits
createCRC32(): Promise<IHasher>
createKeccak(bits?: 224 | 256 | 384 | 512): Promise<IHasher> // default is 512 bits
createMD4(): Promise<IHasher>
createMD5(): Promise<IHasher>
createRIPEMD160(): Promise<IHasher>
createSHA1(): Promise<IHasher>
createSHA224(): Promise<IHasher>
createSHA256(): Promise<IHasher>
createSHA3(bits?: 224 | 256 | 384 | 512): Promise<IHasher> // default is 512 bits
createSHA384(): Promise<IHasher>
createSHA512(): Promise<IHasher>
createXXHash32(seed: number): Promise<IHasher>
createXXHash64(seedLow: number, seedHigh: number): Promise<IHasher>

createHMAC(hashFunc: Promise<IHasher>, key: IDataType): Promise<IHasher>

pbkdf2(
  password: IDataType,
  salt: IDataType,
  iterations: number,
  keyLen: number,
  digest: Promise<IHasher>
): Promise<string>

interface IHasher {
  init: () => void;
  update: (data: IDataType) => void;
  digest: () => string; // returns hash in hex format
  blockSize: number; // in bytes
  digestSize: number; // in bytes
}
```
