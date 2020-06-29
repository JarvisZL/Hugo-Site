# DatastructInproblemsolving


## ADT
- an ADT may be defined as a "class of objects whose logical behavior is defined by a set of values and a set of operations"

## Stack
- a stack is an abstract data type that serves as a collection of elements, with two principal operations:
  - **push**, which adds an element to the collection
  - **pop**, which removes the most recently added element that was not yet removed.
- the strattegy is LIFO(last in, first out).

## heap
- i think we can think of a heap as an almost complete[1] tree that satisfies the heap property:
  - max(min) heap, for any given node C, if P is a parent node of C, then he value)of P is greater(smaller) than or equal to the value of C. 

## queue
-  a queue is a collection of elements with first in first out strategy and that the elements are maintained in a sequence.
-  we can modify a queue by adding elements at one end of the sequence and  removing elements from the other end of the sequence. 

## linked list
-  a linked list is a linear collection of data elements, whose order is not given by their physical placement in memory. Instead, each element points to the next.
-   In its most basic form, each node contains: data, and a reference (in other words, a link) to the next node in the sequence.

## array
- an array is a linear collection of elements, which is stored contiguously in memory. 

## hash table
- a hash table (hash map) is a data structure that can map keys to values.
- A hash table uses a hash function to compute a hash code, into an array, from which the desired value can be found.

## Red-black tree
-  a red–black tree is a kind of self-balancing binary search tree. Each node stores an extra arrtribute representing color, used to ensure that the tree remains approximately balanced during insertions and deletions.
   -  the root node and leaf nodes are black.
   -  if a node is red then its children should be black.
   -  Every path from a given node to leaves should has the same number of black nodes.


## 并查集
- It is a tree-shaped data structure, used to deal with some merge and query problems on disjoint sets. it has two principal operations:
  - Union: Merge two subsets into one set
  - Find: Find the representative element of the set where the given element is located
