---
layout: post
permalink: types
---

My Understanding of types was initially based on structs from c. 
Formalizing my understanding it's like 


- A type is a set of values
- **Example**

**set** | **type**
a set of all integers from -pow(2, 31) to pow(2, 31) - 1 | int
set of true, false | boolean type
x* \| x belongs to chars union epsilon | string

This way of thinking about types I think works in functional paradigm. 

It would help in thinking about composition of functions etc...

**But Object-Oriented Languages require you to think of type in a differently**

- Here Type is thought of as set operations|requests|methods 
end

**we say an object is of type T if the object supports all the operations in T**

So a single object can be many types T1, T2 etc...
I do think this way of understanding types helps to write more naturally in object-oriented languages
I think go defines types this way.

But in c++ this way of thinking won't always work. Due to fact of object slicing etc... but we just limit
to thinking of types this way for purely abstract classes (I mean like an interface in Java in c++ )
This way of reasoning about code certainly helps designing good applications

