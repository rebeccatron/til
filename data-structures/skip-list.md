# Skip List

I learned about skip lists during "algorithms club" with some friends, but couldn't really put it into context. Then, I encountered it again recently through [Bradfield](https://bradfieldcs.com), as part of our study of [LevelDB](https://github.com/google/leveldb/blob/f57513a1d6c99636fc5b710150d0b93713af4e43/db/skiplist.h#L42).

LevelDB uses a skiplist as its "memtable", which means that as new reads and writes come in, LevelDB will use a skip list in-memory to respond to queries. If the skip list fills up

## Definition

So, what is it?

> Skip lists are a data structure that can be used in place of balanced trees. Skip lists use probabilistic balancing rather than strictly enforced balancing and as a result the algorithms for insertion and deletion in skip lists are much simpler and significantly faster than equivalent algorithms for balanced trees.

Per [William Pugh](https://www.epaperpress.com/sortsearch/download/skiplist.pdf).

The coolest thing for me, is the hierarchical structure that emerges, probabilistically:

```text

+-------+                             +--------+                                          +-------+
|       |   LEVEL 2                   |        |                                          |       |
|       +---------------------------->+        +----------------------------------------->+       |
|       |                             |        |                                          |       |
|       |                             |        |                                          |       |
|       |                             |        |                                          |       |
| start |                 +--------+  |        |                             +--------+   | stop  |
|       |   LEVEL 1       |        |  |        |                             |        |   |       |
|       +---------------->+        +->+        +---------------------------->+        +-->+       |
|       |                 |        |  |        |                             |        |   |       |
|       |    +--------+   |        |  |        |   +--------+   +--------+   |        |   |       |
|       | L0 |        |   |        |  |        |   |        |   |        |   |        |   |       |
|       +--->+  k:v   +--->  k:v   +-->  k:v   +-->+  k:v   +-->+  k:v   +--->  k:v   +--->       |
|       |    |        |   |        |  |        |   |        |   |        |   |        |   |       |
+-------+    +--------+   +--------+  +--------+   +--------+   +--------+   +--------+   +-------+

```

So, at the very bottom, in `L0`, every node is present and the structure is basically a linked list.

As we add nodes, we essentially toss a coin to establish the assigned level for the given node. In the case of the 3rd k/v node above, we made it all the way to `L2`. If a node is in Level 2, it'll also be in Levels 1 and 0.

When we want to retrieve a node, we're able to search from the topmost level down, using something that's likely to be akin to binary search. Of course, because the levels are assigned randomly, we may not get the exact behavior of a balanced tree search...but we also don't have the overhead of maintaining the balance of a tree! We just toss coins and snip new nodes in/out as needed.

### Basica Go implementation

```go
package main

import (
	"math/rand"
)

const (
	maxLevels = 12
	branching = 4
)

type (
	skipListOCNode struct {
		item  Item
		next  [maxLevels]*skipListOCNode
		level int
	}

	skipListOC struct {
		head   *skipListOCNode
		levels int
	}

	skipListOCIterator struct {
		o                *skipListOC
		node             *skipListOCNode
		startKey, endKey string
	}
)

func newSkipListOC() *skipListOC {
	head := &skipListOCNode{
		level: maxLevels,
	}
	return &skipListOC{
		head:   head,
		levels: 1,
	}
}

// Find the latest node in each level such that node.item.Key < key, return it as a handy table for future reference
func (o *skipListOC) findNodesBeforeKeyByLevel(key string) [maxLevels]*skipListOCNode {
	var result [maxLevels]*skipListOCNode
	node := o.head
	for i := o.levels - 1; i >= 0; i-- {
		for node.next[i] != nil && node.next[i].item.Key < key {
			node = node.next[i]
		}
		result[i] = node
	}
	return result
}

// dump all the nil checks here, match or bust
func maybeMatch(prev [maxLevels]*skipListOCNode, key string) *skipListOCNode {
	if prev[0].next[0] != nil && prev[0].next[0].item.Key == key {
		return prev[0].next[0]
	}
	return nil
}

func randomLevel() int {
	result := 1
	for result < maxLevels && rand.Intn(branching) == 0 {
		result++
	}
	return result
}

func (o *skipListOC) Get(key string) (string, bool) {
	prev := o.findNodesBeforeKeyByLevel(key)
	if node := maybeMatch(prev, key); node != nil {
		return node.item.Value, true
	}
	return "", false
}

func (o *skipListOC) Put(key, value string) bool {
	prev := o.findNodesBeforeKeyByLevel(key)
	if node := maybeMatch(prev, key); node != nil {
		node.item.Value = value
		return false
	} else {
		level := randomLevel()
		node := &skipListOCNode{
			item:  Item{Key: key, Value: value},
			level: level,
		}
		if level > o.levels {
			for i := level - 1; i >= o.levels; i-- {
				o.head.next[i] = node
			}
			for i := o.levels - 1; i >= 0; i-- {
				node.next[i] = prev[i].next[i]
				prev[i].next[i] = node
			}
			o.levels = level
		} else {
			for i := level - 1; i >= 0; i-- {
				node.next[i] = prev[i].next[i]
				prev[i].next[i] = node
			}
		}
		return true
	}
}

func (o *skipListOC) Delete(key string) bool {
	prev := o.findNodesBeforeKeyByLevel(key)
	if node := maybeMatch(prev, key); node != nil {
		for i := node.level - 1; i >= 0; i-- {
			prev[i].next[i] = node.next[i]
		}
		return true
	} else {
		return false
	}
}

func (o *skipListOC) RangeScan(startKey, endKey string) Iterator {
	prev := o.findNodesBeforeKeyByLevel(startKey)
	node := prev[0].next[0]
	return &skipListOCIterator{o, node, startKey, endKey}
}

func (iter *skipListOCIterator) Next() {
	iter.node = iter.node.next[0]
}

func (iter *skipListOCIterator) Valid() bool {
	return iter.node != nil && iter.node.item.Key <= iter.endKey
}

func (iter *skipListOCIterator) Key() string {
	return iter.node.item.Key
}

func (iter *skipListOCIterator) Value() string {
	return iter.node.item.Value
}

```

In terms of performance, the skip list is a bit worse than a red/black tree, but it should be significantly better than a linked list for most operations.

```console
Testing using 10000 words (random pattern)

Name                     Puts                Deletes             Gets                RangeScan
----------------------------------------------------------------------------------------------------
Slice                    1.356903122s        11.193689ms         2.392295ms          38.957µs
Linked List              333.221781ms        274.716275ms        374.140125ms        60.419µs
Linked Block             107.781018ms        2.007883ms          2.769629ms          40.28µs
Binary Search Tree       3.537163ms          1.707302ms          2.950262ms          271.332µs
Red Black Tree           2.85031ms           1.314272ms          2.789122ms          193.902µs
Skip List                4.969778ms          1.88451ms           3.389008ms          49.639µs
```