---
title: Recursive Generator Functions
slug: recursive-generator-functions
date_published: 2021-10-04T18:03:51.000Z
date_updated: 2021-10-04T18:03:51.000Z
tags: python, programming, generator, recursion, binarytree
---

You can find previous post about Python generator function [here ](/2021-09-19-python-iterators/)

This is something I haven't used very often but came up in a discussion. 

We know the generator function and recursive function. How do we create a generator function which is recursive.

For example, if we have a binary tree and performing in-order traversal. A recursive function looks like this -

```
    l = []
    def in_order(root):
    	if root:
        		in_order(root.left)
            	l.append(root)
            	in_order(root.right) 
         
    in_order(root)
    for node in l:
    	do_some_thing(node)
```

This is a standard implementation. You get all the elements of tree in a list and then iterate over that .

To convert this to a generator function, we have to use *yield *with the current value of node (the one which we want to return).

```
	def in_order(root):
	    	if root:
	        		in_order(root.left)
	            	yield(root)
	            	in_order(root.right)
	
```
Once we use yield in this function, the return type of this function changes to a generator function. That means *in_order(root) *will return an generator object and to get a value out of that, we have to iterate over that .

```
	def in_order(root):
	    	if root:
	        		in_order(root.left)
	            	yield(root)
	            	in_order(root.right)
	            
	    for node in in_order(root):
	     	do_some_thing(node)
```

But this change alone won't make it work. If you see, this recursive function internally use  *in_order(root.left) and in_order(root.right) *which again is going to return a generator. Hence to actually do some thing in these recursive calls, we will have to use yield in those calls as well.

```
	def in_order(root):
	    	if root:
	                for node in in_order(root.left):
	                    yield node
	                yield(root)
	                for node in in_order(root.right):
	                    yield node
	            
	     for node in in_order(root):
	     	do_some_thing(node)
```

This makes it a recursive generator function. The simple rule is - use yield with the value that we want to return and and since recursive calls also returns generator function, use a for loop - yield to get the value from those .

---
