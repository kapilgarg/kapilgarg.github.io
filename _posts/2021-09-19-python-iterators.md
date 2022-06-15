---
title: Python Generator
slug: python-iterators
date_published: 2021-09-19T17:32:59.000Z
date_updated: 2021-09-19T17:32:59.000Z
tags: python, generator, programming
---

When you use a loop to iterate over a collection of items (like list), this collection/sequence is stored in memory. For small sequences, this approach works well. but If you have to iterate over a large sequence (may be millions of records), then the memory usage may be much higher which may not be required.

> nums = [1,2,3,4,...5000000]  
> for i in nums:print(i,end=',')  
> ...  
> 1,2,3,4,...5000000,>>>  

All we are trying to do here is read a number from the *nums *and performs *some action* on that. 

Python provides a way where you could generate the next element in sequence whenever you need. That means, you could generate 1,2,3,4,...5000000 as in previous example without actually storing these numbers in the list.

The object that generates the next number in sequence is called '*generator*'. You could use this object in the same way (in for loop) as you are using the list. Generator does not store *complete sequence,* instead it computes the next item in sequence and returns it

For example, lets write a simple generator object to generate number from 1 to n

```
    def num_generator(n):
    """
    returns a generator object which generates values from 1 to n
    """
            for i in range(1,n+1):
                    yield i

            
    >>> nums = num_generator(5)
    
   
    # using generator object like we use a list in a loop
    >>> for i in nums:
    ...     print(i,end=',')
    ...
   
    1,2,3,4,5,>>>

```

We have defined a function *num_generator *which returns a generator object. In this function '*yield*' is the key word which converts the output of this function to a generator object . This generator object can be used in a loop in the same way we used any list.

*When you call a function containing yield, the code in the function body does not run. The function returns a generator object. Whenever you read an items from the generator object, python execute the function until it reaches yield statement, gives you the item and pause. next time, when you read another item, it resume the function and and returns you another object. It continues doing that till there are no more objects to generate. At that point, loop is over.*

### So if this is the case, why not always use generator ?

Generators have some *limitations*.  With generator, you could only move in *forward* direction. At any point in time, if you have to read the previous value, generator can not provide that.

In out example, say, after reaching 4, if we need to look the previous value i.e 3, generator can not do that. That's why these should be used with care.

Generators are useful where the you need to iterate over a large sequence and 

1. You don't need to go back and forth with the values in the sequence. 
2. Depending on some criteria, you may terminate the iteration hence you don't want to store all the items in memory.
3. You could generate the value *locally*.

---
