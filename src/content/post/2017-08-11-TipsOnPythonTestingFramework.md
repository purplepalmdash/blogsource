+++
title = "TipsOnTesingFrameWorkForPython"
date = "2017-08-11T14:27:43+08:00"
description = "tips on testingframework for python"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Code
Before you run:    

```
# apt-get install python-pip
# pip install nose
# pip install python-memcached
```

Directory Structure:      

```
test@local:~$ tree /tmp/foomodule/
/tmp/foomodule/
|-- foo
|   |-- a.py
|   |-- b.py
|   `-- __init__.py
`-- tests
    |-- test_a.py
    `-- test_b.py
```
Module Source Code:    

```
# /tmp/foomodule/foo/a.py 

def add(a, b):
    return a + b
def double(a):
    return a * 2

# /tmp/foomodule/foo/b.py 

import memcache
class Cache:
    def __init__(self, server):
        self.cache = memcache.Client([server])
    def get(self, name):
        return self.cache.get(name)
    def set(self, name, value):
        return self.cache.set(name, value)
    def delete(self, name):
        return self.cache.delete(name)
    def close(self):
        self.cache.disconnect_all()

# /tmp/foomodule/foo/__init__.py

from a import *
from b import *
```
Testing Code:    

```
# /tmp/foomodule/tests/test_a.py 
from foo.a import add, double
def test_add():
    v = add(10, 20)
    assert v == 30
def test_double():
    v = double(10)
    assert v == 20

# /tmp/foomodule/tests/test_b.py 
from foo.b import Cache
class TestCache:

    def setUp(self):
        self.cache = Cache("172.17.0.1:11211")
        self.key   = "name"
        self.value = "smallfish"

    def tearDown(self):
        self.cache.close()

    def test_00_get(self):
        v = self.cache.get(self.key)
        assert v == None

    def test_01_set(self):
        v = self.cache.set(self.key, self.value)
        assert v == True
        v = self.cache.get(self.key)
        assert v == self.value

    def test_02_delete(self):
        v = self.cache.delete(self.key)
        assert v == True
```
memcached running in docker:    

```
# docker pull memcached
# docker run -d -p 11211:11211 memcached
```
Run testing via:    

```
# nosetests -s -v
test_a.test_add ... ok
test_a.test_double ... ok
test_b.TestCache.test_00_get ... ok
test_b.TestCache.test_01_set ... ok
test_b.TestCache.test_02_delete ... ok

----------------------------------------------------------------------
Ran 5 tests in 0.012s

OK
```
By default, nose will run test to `test` directory, file(contains test),
functions(started with test_), class(started with Test), assert will run
assertion.    

### coverage
Install coverage via:    

```
# pip install coverage
```

Run with coverage:    

```
# coverage html
# ls
foo  htmlcov  tests
# nosetests -s -v --with-coverage
# coverage report -m
```
nosetests could be run with coverage items.    
