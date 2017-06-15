PyQ provides seamless integration of Python and q code. It brings Python and q interpreters in the same process and allows code written in either of the languages to operate on the same data. In PyQ, Python and q objects live in the same memory space and share the same data.  
<i class="fa fa-hand-o-right"></i> [pyq.enlnt.com](http://pyq.enlnt.com) 


## Features

The following annotated session demonstrates some of the features:

- start interactive session and import the `q` handle:
    ```bash
    $ pyq
    >>> from pyq import q
    ```
- create an empty table
    ```python
    >>> q.trade = q('([]date:();sym:();qty:())')
    ```
- insert data
    ```python
    >>> q.insert('trade', (date(2006,10,6), 'IBM', 200))
    >>> q.insert('trade', (date(2006,10,6), 'MSFT', 100))
    ```
- display the result
    ```python
    >>> q.trade.show()
    date       sym  qty
    -------------------
    2006.10.06 IBM  200
    2006.10.06 MSFT 100
    ```
- define a parameterized query
    ```python
    >>> query = q('{[s;d]select from trade where sym=s,date=d}')
    ```
- run a query
    ```python
    >>> query('IBM', date(2006,10,6))
    k('+`date`sym`qty!(,2006.10.06;,`IBM;,200)')
    ```
- pretty-print the result
    ```python
    >>> q.show(_)
    date       sym  qty
    ------------------
    2006.10.06 IBM 200
    ```


## Installation

```bash
$ pip install -i https://pypi.enlnt.com --no-binary pyq pyq
```
Use and redistribution are subject to a [license](https://pyq.enlnt.com/license.html).


## On-line documentation

Documentation is available from the Python prompt
```python
>>> help('pyq')
```
or at the [PyQ homepage](https://pyq.enlnt.com).


## Performance 

Q is much faster than Python. The following table shows times of adding 10,000 integers in four different ways: 

1. using a q function on q data
2. using a q function on Python data
3. using a Python function on Python data
4. using a Python function on q data

| Setup              | Call       | Time      |
|--------------------|------------|-----------|
| `x = q.til(10000)` | `q.sum(x)` | 31.5 usec |
| `x = range(10000)` | `q.sum(x)` | 187 usec  |
| `x = range(10000)` | `sum(x)`   | 648 usec  |
| `x = q.til(10000)` | `sum(x)`   | 849 usec  |


### One-way conversion

The best performance is achieved when q functions are used on q data. PyQ is designed to take advantage of this fact. Python data is automatically converted to q, when mixed with q data and stays in q form unless converted back explicitly:
```python
>>> q('1 2 3') + [3,2,1]
k('4 4 4')
>>> list(_)
[4, 4, 4]
```


### Compiled queries

Parsing large queries can be slow. PyQ makes it easy to pre-compile q queries and reuse them as if they were Python functions.
```python
>>> query = q('{[s;d]select from trade where sym=s,date=d}')
```
PyQ queries combine the features of Python and q functions. Like Python functions, they can be called with named arguments.
```python
>>> query(d=date(2006,10,6),s='IBM')
k('+`date`sym`qty!(,2006.10.06;,`IBM;,200)')
```
and, like q functions, they can be partially applied:
```python
>>> q1 = query(d=date(2006,10,6))
```
resulting in a query with fewer parameters.
```python
>>> q1('MSFT')
k('+`date`sym`qty!(,2006.10.06;,`MSFT;,100)')
```


## Writing a client

PyQ is designed primarily for embedded in-memory databases. For best performance one should avoid sending data between processes or over a network. Nevertheless, client programming is very simple with PyQ.
```python
>>> from qc import *
>>> c = Client(port=1234)
>>> c.insert('t', (1,'x'))
>>> c.insert('t', (2,'y'))
>>> c.insert('t', (3,'z'))
>>> q.show(c('t'))
a b
---
1 x
2 y
3 z
>>> c('{select from t where a>x}'), 1)
k('+`a`b!(2 3;`y`z)')
```

