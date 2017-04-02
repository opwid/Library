
# Maps

A map is an abstract data type designed to efficiently store and retrieve values based upon a uniquely identifying search key for each. Specifically, a map stores key-value pairs (k, v), which we call entries, where k is the key and v is its corresponding value. Keys are required to be unique, so that the association of keys to values defines a mapping.
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
