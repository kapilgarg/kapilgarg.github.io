---
title: Create and initialize a list in python
slug: create-and-initialize-a-list
date_published: 2020-06-16T17:28:17.000Z
date_updated: 2020-06-16T17:28:17.000Z
tags: python, tips, programming, pythonlist, list comprehension
---

Create a list and initialize it with some default values.

    #create a list of 10 elements with default value as 0
    >>> my_list = [0]*10
    >>> my_list
    [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Later you can assign the value to individual elements 

    >>> my_list[0]=100
    >>> my_list
    [100, 0, 0, 0, 0, 0, 0, 0, 0, 0]

Above approach works if the contents of the list are immutable. If the list contents are mutable or if the default value is an object type, this approach does not work. In that case, updating a single element would reflect same change in all the elements.

For example, *creating a list of dictionary and initializing it with empty dictionaries*. 

Whenever any individual dictionary is updated, that results in updating all the dictionaries this list contains. The reason for this is same reference that all the items in the list contains.

    >>> my_list = [{}]*10
    >>> my_list
    [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}]
    
    >>> my_list[0]['key']='value'
    
    >>> my_list
    [{'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}, {'key': 'value'}]

To handle this use case, use python list comprehension

    >>> my_list = [{} for _ in range(10)]
    >>> my_list
    [{}, {}, {}, {}, {}, {}, {}, {}, {}, {}]
    >>> my_list[0]['key']='value'
    >>> my_list
    [{'key': 'value'}, {}, {}, {}, {}, {}, {}, {}, {}, {}]

happy coding ...
