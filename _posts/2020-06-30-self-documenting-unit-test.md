---
title: Self documenting unit test
slug: self-documenting-unit-test
date_published: 2020-06-30T17:48:07.000Z
date_updated: 2020-06-30T17:48:07.000Z
tags: python, unittest, pythontips
---

When you write unit test case with python unittest module and run the test,          by default it prints the *testcase name* (*module name)...status.*

    test_is_even_number (test_math_util.TestMathUtil) ... ok
    
    ----------------------------------------------------------------------
    Ran 1 test in 0.002s
    
    OK

If you have a couple of testcases for a given function, these print statement will not provide much information as to which test case is successful or failed.

What if these tests are self documenting . When they run, they document which testcase is successful / failed. something similar to this -

    test_is_even_number (test_math_util.TestMathUtil)
    
                    test for odd number
                     ... ok
    test_is_even_number_2 (test_math_util.TestMathUtil)
    
                    test for even number
                     ... ok
    
    ----------------------------------------------------------------------
    Ran 2 tests in 0.003s
    
    OK

Here, I ran two test for even and odd number and the log statement shows the testcase for which it failed or successful. This may be helpful since by looking at the log statement, you can find out what happened.

### How to do this

In your test class, you *just* need to override shortDescription method to return *_testMethodDoc *as given in the example below. that's all is required.

    def is_even_number(number):
    	return number%2 == 0

math_util.py
    import unittest
    import math_util
    
    class TestMathUtil(unittest.TestCase):
    	def shortDescription(self):
    		return self._testMethodDoc
    
    	
    	def test_is_even_number(self):
    		"""
    		test for odd number
    		"""
    		num=5
    		actual = math_util.is_even_number(num)
    		self.assertFalse(actual)
            
    	def test_is_even_number_2(self):
    		"""
    		test for even number
    		"""
    		num=10
    		actual = math_util.is_even_number(num)
    		self.assertTrue(actual)

test_math_util.py
further reading - [https://pythonhosted.org/gchecky/unittest-pysrc.html](https://pythonhosted.org/gchecky/unittest-pysrc.html)
