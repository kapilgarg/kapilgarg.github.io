While writing test cases, you may need to mock one or more environment variables. It could be because environment variable it used as a flag for certain functionality or something else.

Environment variables are available through os.environ api. Though technically os.environ is not a dictionary but it fulfills the criteria so that it can be mocked like a dictionary. unittest module provides a way to achieve this.

mock.patch.dict is the function that we can use to mock a dictionary. First argument is the target dictionary that you want to mock, 2nd argument is the value (dictionary) that you want your target dictionary to return.  
```python
import os
from unittest import TestCase, mock

class FileWriterTests(TestCase):
    @mock.patch.dict(os.environ, {"DESTINATION": "AWS"}, clear=True)
    def test_write_file(self):        
        self.assertEqual(os.environ.get('DESTINATION'), 'AWS')
```

The dictionary mocked here will be available through out the test . 'clear=True' will make sure that dictionary is restored to its original state when the test ends.

You could also mock this dictionary at class level so that it is available for all the tests.  

```python
import os
from unittest import TestCase, mock

@mock.patch.dict(os.environ, {"DESTINATION": "AWS"}, clear=True)
class FileWriterTests(TestCase):    
    def test_write_file(self):        
        self.assertEqual(os.environ.get('DESTINATION'), 'AWS')
        
    def test_read_file(self):        
        self.assertEqual(os.environ.get('DESTINATION'), 'AWS')
```

When setting the values in the dictionary, you could also use key words.

```python
with patch.dict('os.environ', DESTINATION='AWS'):
...     print(os.environ['DESTINATION'])
```

Please note that when you patch dict at class level, patched dictionary is only available to test methods starting with 'test_'. This will not be avialable in setUp, tearDown() etc.

You could also use a function to provide the value for key. If the values of the dictionary are dynamic, you could use context manager to provide the dynamic values 

```python
import os
from unittest import TestCase, mock

class FileWriterTests(TestCase):    
    def test_write_file(self):
    	with patch.dict('os.environ', {"DESTINATION": get_destination()}):
	     	print(os.environ['DESTINATION'])
        	self.assertEqual(os.environ.get('DESTINATION'), 'AWS')
```
Hope this helps.

Happy coding ...

ref : https://docs.python.org/3/library/unittest.mock.html#patch-dict

[1] https://github.com/python/cpython/blob/4d95fa1ac5d31ff450fb2f31b55ce1eb99d6efcb/Lib/os.py#L665

[2] patch.dict() can be used with dictionary like objects that aren’t actually dictionaries. At the very minimum they must support item getting, setting, deleting and either iteration or membership test. This corresponds to the magic methods __getitem__(), __setitem__(), __delitem__() and either __iter__() or __contains__().