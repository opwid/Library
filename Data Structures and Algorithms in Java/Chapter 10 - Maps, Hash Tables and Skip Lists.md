
# Maps

A map is an abstract data type designed to efficiently store and retrieve values based upon a uniquely identifying search key for each. Specifically, a map stores key-value pairs (k, v), which we call entries, where k is the key and v is its corresponding value. Keys are required to be unique, so that the association of keys to values defines a mapping.  

On a map with n entries, each of the fundamental methods takes O(n) time in the worst case because of the need to scan through the entire list when searching for an existing entry. Fortunately, as we discuss in the next section, there is a much faster strategy for implementing the map ADT.  


## The Map ADT 


| Method      | Description                                                                                                                                                                                        |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| size():     | Returns the number of entries in M.                                                                                                                                                                |
| isEmpty():  | Returns a boolean indicating whether M is empty.                                                                                                                                                   |
| get(k):     | Returns the value v associated with key k, if such an entry exists; otherwise returns null.                                                                                                        |
| put(k,v):   | If M does not have an entry with key equal to k, then adds entry(k, v) to M and returns null; else, replaces with v the existing value of the entry with key equal to k and returns the old value. |
| remove(k):  | Removes from M the entry with key equal to k, and returns its value; if M has no such entry, then returns null.                                                                                    |
| keySet():   | Returns an iterable collection containing all the keys stored in M.                                                                                                                                |
| values():   | Returns an iterable collection containing all the values of entries stored in M (with repetition if multiple keys map to the same value).                                                          |
| entrySet(): | Returns an iterable collection containing all the key-value entries in M.                                                                                                                          |


# Hash Tables

In this section, we introduce one of the most efficient data structures for implementing a map, and the one that is used most in practice. This structure is known as a hash table.  

Intuitively, a map M supports the abstraction of using keys as "addresses" that help locate an entry. The novel concept for a hash table is the use of a hash function to map general keys to corresponding indices in a table. Ideally, keys will be well distributed in the range from 0 to N − 1 by a hash function, but in practice there may be two or more distinct keys that get mapped to the same index. As a result, we will conceptualize our table as a bucket array in which each bucket may manage a collection of entries that are sent to a specific index by the hash function.

## Hash Function

The goal of a hash function, h, is to map each key k to an integer in the range [0, N − 1], where N is the capacity of the bucket array for a hash table. Equipped with such a hash function, h, the main idea of this approach is to use the hash function value, h(k), as an index into our bucket array, A, instead of the key k (which may not be appropriate for direct use as an index). That is, we store the entry(k, v) in the bucket A[h(k)].  

If there are two or more keys with the same hash value, then two different entries will be mapped to the same bucket in A. In this case, we say that a collision has occurred. We say that a hash function is "good" if it maps the keys in our map so as to sufficiently minimize collisions. For practical reasons, we also would like a hash function to be fast and easy to compute.  

It is common to view the evaluation of a hash function, h(k), as consisting of two portions—a hash code that maps a key k to an integer, and a compression function that maps the hash code to an integer within a range of indices, [0, N − 1], for a bucket array.


## Compression Function

The hash code for a key k will typically not be suitable for immediate use with a bucket array, because the integer hash code may be negative or may exceed the capacity of the bucket array. Thus, once we have determined an integer hash code for a key object k, there is still the issue of mapping that integer into the range [0, N −1]. This computation, known as a compression function, is the second action performed as part of an overall hash function. A good compression function is one that minimizes the number of collisions for a given set of distinct hash codes. 

## Collusion-Handling Schemes
### Separate Chaining 
A simple and efficient way for dealing with collisions is to have each bucket A[j] store its own secondary container, holding all entries (k, v) such that h(k) = j. A natural choice for the secondary container is a small map instance implemented using an unordered list.  
In the worst case, operations on an individual bucket take time proportional to the size of the bucket. Assuming we use a good hash function to index the n entries of our map in a bucket array of capacity N, the expected size of a bucket is n/N. Therefore, if given a good hash function, the core map operations run in O(⌈n/N⌉). The ratio λ = n/N, called the __load factor__ of the hash table, should be bounded by a small constant, preferably below 1. As long as λ is O(1), the core operations on the hash table run in O(1) expected time. 

### Open Addressing
The separate chaining rule has many nice properties, such as affording simple implementations of map operations, but it nevertheless has one slight disadvantage: It requires the use of an auxiliary data structure to hold entries with colliding keys. Open addressing requires that the load factor is always at most 1 and that entries are stored directly in the cells of the bucket array itself. 

### Linear Probing

A simple method for collision handling with open addressing is linear probing. With this approach, if we try to insert an entry (k,v) into a bucket A[j] that is already occupied, where j = h(k), then we next try A[(j + 1) mod N]. If A[(j + 1) mod N] is also occupied, then we try A[(j + 2) mod N], and so on, until we find an empty bucket that can accept the new entry. Once this bucket is located, we simply insert the entry there. Of course, this collision resolution strategy requires that we change the implementation when searching for an existing key—the first step of all get, put, or remove operations. In particular, to attempt to locate an entry with key equal to k, we must examine consecutive slots, starting from A[h(k)], until we either find an entry with an equal key or we find an empty bucket.  

To implement a deletion, we cannot simply remove a found entry from its slot in the array. A typical way to get around the difficulty is to replace a deleted entry with a special "defunct" sentinel object. With this special marker possibly occupying spaces in our hash table, we modify our search algorithm so that the search for a key k will skip over cells containing the defunct sentinel and continue probing until reaching the desired entry or an empty bucket (or returning back to where we started from). Additionally, our algorithm for put should remember a defunct location encountered during the search for k, since this is a valid place to put a new entry (k, v), if no existing entry is found beyond it.  

Another open addressing strategy, known as quadratic probing, iteratively tries the buckets A[(h(k) + f (i)) mod N], for i = 0,1,2,..., where f (i) = i<sup>2</sup>, until finding an empty bucket. As with linear probing, the quadratic probing strategy complicates the removal operation, but it does avoid the kinds of clustering patterns that occur with linear probing. Nevertheless, it creates its own kind of clustering, called secondary clustering, where the set of filled array cells still has a nonuniform pattern, even if we assume that the original hash codes are distributed uniformly.  

An open addressing strategy that does not cause clustering of the kind produced by linear probing or the kind produced by quadratic probing is the double hashing strategy. In this approach, we choose a secondary hash function, h′, and if h maps some key k to a bucket A[h(k)] that is already occupied, then we iteratively try the buckets A[(h(k) + f (i)) mod N] next, for i = 1,2,3,..., where f (i) = i·h′(k).  

![10.2](https://github.com/opwid/Library/blob/master/Data%20Structures%20and%20Algorithms%20in%20Java/Images/10.2.png)  

# Sorted Maps
The traditional map ADT allows a user to look up the value associated with a given key, but the search for that key is a form known as an exact search. In this section, we will introduce an extension known as the sorted map ADT. 

## Sorted Search Tables

Several data structures can efficiently support the sorted map ADT, and we will examine some advanced techniques. In this section, we will begin by exploring a simple implementation of a sorted map. We store the map's entries in an array list A so that they are in increasing order of their keys. We refer to this implementation as a sorted search table.  

As was the case with the unsorted table map of Section 10.1.4, the sorted search table has a space requirement that is O(n). The primary advantage of this representation, and our reason for insisting that A be array-based, is that it allows us to use the binary search algorithm for a variety of efficient operations.  

It should be clear that the size, firstEntry, and lastEntry methods run in O(1) time, and that iterating the keys of the table in either direction can be performed in O(n) time. The analysis for the various forms of search all depend on the fact that a binary search on a table with n entries runs in O(log n) time.












