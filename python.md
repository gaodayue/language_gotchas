python gotchas
====
------


modify variable in outer function
----
```python
def outer():
    times_called = 0
    def inner():
        times_called += 1
        print "called", times_called, "times"
    return inner

f = outer()
f() # raise an UnboundLocalError
```

Reason: Variable assignment `times_called += 1` declares a new variable (or *name* in python) `time_called` in the **scope** of `inner` function.

Solution:

1. wrap variables to be modified in a list

        def outer():
            times_called = [0]
            def inner():
                times_called[0] += 1
                print "called", times_called[0], "times"
            return inner


2. use class instead

        class Counter(object):
            def __init__(self):
                self.times_called = 0
            def __call__(self):
                self.times_called += 1
                print "called", self.times_called, "times"
        
        f = Counter()
        f(); f(); f()

3. In python3, use `nonlocal` keyword

        def outer():
            times_called = 0
            def inner():
                nonlocal times_called # means "don't create new variable, use outer one"
                times_called += 1
                print "called", times_called, "times"
            return inner


<a id="default_argument"></a>
default argument
----

Default argument is evaluated when function is defined, not when function is executed!

```python
>>>import time
>>> def report(when=time.time()):
...     print when
...
>>> report()
1210294387.19
>>> time.sleep(5)
>>> report()
1210294387.19
```

Another example,

```python
>>> def foo(x=[]):
...     x.append(1)
...     print x
... 
>>> foo()
[1]
>>> foo()
[1, 1]
>>> foo()
[1, 1, 1]
```

Solution, use None for default argument value instead:

```python
>>> def foo(x=None):
...     if x is None:
...         x = []
...     x.append(1)
...     print x
```


closure closes over variables, not values
----

```python
funcs = []
for i in range(4):
    def f():
        print i
    funcs.append(f)

for f in funcs:
    f() # print four '3'
```

Reason: `for i in range(4)` declares a variable `i` and assigns to it different value each iteration. But **closure closes over variable i, not its value**. So when `f` is executed, it looks up `i`'s value in the environment, which is 3.

Solution:

1. use a factory function.

        def make_f(i):
            def f():
                print i
            return f
        
        funcs = []
        for i in range(4):
            funcs.append(make_f(i))

2. another way depends on python's [default argument](#default_argument) behavior, but ugly.

        funcs = []
        for i in range(4):
            def f(i=i):
                print i
            funcs.append(f)