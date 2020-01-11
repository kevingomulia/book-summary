# Chapter 10 - Collections - II

## 10.4 Understanding the Performance of Collections
In many cases, you can reason about the performance of a collection by understanding its basic structure. For instance, a `List` is a singly linked list. It’s not indexed, so if you need to access the one-millionth element of a List as `list(1000000)`, that will be slower than accessing the one-millionth element of an `Array`, because the `Array` is indexed, whereas accessing the element in the List requires traversing the length of the `List`.

| Key | Description |
| --- | --- |
| C | The operation takes constant time |
| eC | The operation takes effectively constant time, but this might depend on some assumptions, such as maximum length of a vector, or distribution of hash key |
| aC | The operation takes amortized constant time. |
| Log | The operation takes time proportional to the logarithm of the collection size |
| L | The operation is linear - proportional to the collection size |
| - | The operation is not supported |
