# Merkle Hash Trees

A merkle tree is a tree, such that:
* Every leaf is labeled with a cryptographic hash
* Ever non-leaf node is labeled with the cryptographic hash of *all* its child nodes

```text
                              +---------------------+                          digests of all digests
                              |  h(1,2,3,4,5,6,7,8) |
                              +----------^----------+
                                         |
                                         |
                                         |
                 +-------------+---------|----------+-------------+            digest of digests
                 |  h(1,2,3,4) |                    | h(5,6,7,8)  |
                 |             |                    |             |
                 +------^------+                    +-------^-----+
                        |                                   |
                        |                                   |
        +-------------+ + +-------------+   +-------------+ + +-------------+  digest of digests
        |    h(1, 2)  |   |   h(3, 4)   |   |   h(5, 6)   |   |   h(7, 8)   |
        |             |   |             |   |             |   |             |
        +------^------+   +------^------+   +------^------+   +------^------+
               |                 |                 |                 |
               |                 |                 |                 |
        +------++-----+   +------+  ----+   +------+ -----+   +------+ -----+   digests of content
        | h(1)| | h(2)|   | h(3)| | h(4)|   | h(5)| | h(6)|   | h(7)| | h(8)|
        |     | |     |   |     | |     |   |     | |     |   |     | |     |
        +--^--+ +--^--+   +--^--+ +--^--+   +--^--+ +--^--+   +--^--+ +--^--+
           |       |         |       |         |       |         |       |
           |       |         |       |         |       |         |       |
           |       |         |       |         |       |         |       |
        +--+--+ +--+--+   +--+--+ +--+--+   +--+--+ +--+--+   +--+--+ +--+--+   raw content
        |  1  | |  2  |   |  3  | |  4  |   |  5  | |  6  |   |  7  | |  8  |
        |     | |     |   |     | |     |   |     | |     |   |     | |     |
        +-----+ +-----+   +-----+ +-----+   +-----+ +-----+   +-----+ +-----+

```

Patented in 1982 by Ralph C. Merkle, [here](https://patents.google.com/patent/US4309569). 

Merkle trees offer efficient, secure verification of _all content_ in a given data structure...how? The structure above means that the ur-parent node, the root, contains the hashes of everything else in the structure. This "root" hash can be shared, along with paths along the tree from specific leaves up to the root. To verify the content of all data, or just a subset of the data, the receiver can simply compute the hashing of content (or a path within the content). If the root hash doesn't match, the data has been tampered with or is otherwise incorrect.

Think of them as a "cryptographic accumulator"! Arbitrarily many chunks of data (and their associated, individual hashesh), sort of "roll up" into a single hash, which can easily be shared and re-computed by a receiver.
* `hash(hash(1) + hash(2))`
* hashing has to happen in-order (via something like concatenation, NOT something commutative like `XOR`)

MVP sample code, based on example from [here](https://nakamoto.com/merkle-trees/):

```python
from hashlib import sha1 #sha1 is broken, but w/e

def hash(input):
   return sha1(input.encode()).hexdigest()

if __name__ == '__main__':
   stuff1 = "my stuff!"
   stuff2 = "more stuff!"

   digest1 = hash(stuff1)
   digest2 = hash(stuff2)

   root = hash(digest1 + digest2) # will uniquely identify all the stuff
   print(root) # 5dc6bff1828e2ee54a05b194cda4b8e03cf32623 or something similar
```

Merkle trees also allow for quick **inclusion proofs**. Given the following:
1. The content / stuff we want to prove
1. The canonical merkle root
1. All "sibling" node digests along the path "up" from the content --> the root

To test inclusion, I can re-hash my content (#1) in the standard way. Then, iteratively, the digests at each layer can be re-computed by combining the OG content digest + its appropriate sibling digests. If any re-generated digest along the path to the root disagrees with the provided values, I've found the corrupted node.

How many hashes need to be exchanged for this verification? On the order of `O(log(n))`.

## Foundational Concepts

Merkle Hash Trees are built on a few other core concepts:
* Binary Trees
* Hash Functions

### Hashing and Hash Functions

Hash functions deterministically map arbitrary input into fixed output. Crucially, *cryptographic* hash functions are one-way functions; so, we should not be able to take the output and reconstruct the input that produced it. Also, a NTH for cryptographic hashes is the avalanche effect: if I change even a single bit in the input, lots of changes should trigger in the output. Also also, we want to avoid "collisions" - multiple inputs mapping to the same outputs. See [here](https://crypto.stackexchange.com/questions/13299/is-80-bits-of-key-size-considered-safe-against-brute-force-attacks/13302#13302) for more on collision spaces.

Note: a hash function's "input" is commonly called a "preimage" and the output is called a "digest".

Common Hash Functions:
* SHA-2
   * "Secure Hash Algorithm 2"
   * [Thanks, NSA!](https://www.algorithmhalloffame.org/algorithms/sha-2/)
   * There are 224, 256, 384, and 512 bit versions. That is, the number is the length of the output "digest".
   * Used in TLS, SSL, PGP, SSH, an other security applications/protocols.
* MD5
   * "Message Digest, v5"
   * Outputs a 128-bit hash value
   * Originally a cryptographic hash but has well-known security issues so best used for non-security crucial cases (e.g. [partition keys](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecordsRequestEntry.html), like in Kinesis)
* [CRC32](http://ross.net/crc/download/crc_v3.txt)
   * NOT cryptographic...these are easily reversible!
* Can play with others [here](https://emn178.github.io/online-tools/crc32.html).

Good hash functions always produce the same digest for a given preimage; have fixed output sizes; produce uniformly distributed digests, sprinkled evenly across the output space.

The simplest integer hash function that satisfies these "good" requirements might be:

```python

def hash(input_number):
   return input_number % 255
```

## Resources

* [Intro to Cryptocurrency](https://nakamoto.com/introduction-to-cryptocurrency/) 