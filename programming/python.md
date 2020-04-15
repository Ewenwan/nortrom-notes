
[toc]

[programming](./prog.md)

# basic concept

* difference between package and module
    * A module is a single file (or files) that are imported under one import and used.
        * `import my_module`
    * A package is a collection of modules in directories that give a package hierarchy.
        * `from my_package.timing.danger.internets import function_of_love`

# common opeartion

## loop

* assign value in a loop in one line

    ```python
    squares = [x**2 for x in range(10)]
    [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
    ```

## string

### simple string ops

* Reverse String
    * `s[::-1]`
* Format print
    * `print ("His name is %s"%("Aviad"))`
* Format a string
    * `"xxxx{:x}xxx{:}".format(hexV1, v2)`
    * Or just the same as print
    * `str += ("%s" %(…))`
* String alignment
    * Left adjustment or right adjustment(左对齐右对齐)
    * `str.ljust(10,'+')  str.rjust(10,'+')`
    * '+', means complement by char '+'
* Remove space char in beginning and ending
	* `string.lstrip(s[, chars])`
	* `string.rstrip(s[, chars])`

### json

* convert json to dict object
    * method 1

    ```python
    with open("../config/record.json","w") as f:
    json.dump(new_dict,f)
    ```

    * method 2

    ```python
    x = json.loads(data, object_hook=lambda d: namedtuple('X', d.keys())(*d.values()))
    ```

    * 参考：[How to convert JSON data into a Python object](https://stackoverflow.com/questions/6578986/how-to-convert-json-data-into-a-python-object)

### re

* Remove `^M(\r)` and other special char in string:
    * terminal `cat file | tr -s "\r" "\n" > new_file`
    * vim `%s/\r//gc`
    * python `v=re.sub('[\r\n\t]','',v)`
* Extract information from pattern
    * `(?<=…)xxx`  before match, match `…xxx`, return `xxx`
    * `xxx(?=…)` after match, match `xxx…`, return `xxx`
    * `\w+`, matching number|chars
    * `[\w+,]*` , matching number|chars|, for multiple times
* Re options
    * `re.I(re.IGNORECASE)`: 忽略大小写
    * `re.S(DOTALL)`: 点任意匹配模式
    * `re.M(MULTILINE)`: 多行模式


## list/set/dict

* list append and extend
    * append: Appends object at end.

    ```python
    x = [1, 2, 3]
    x.append([4, 5])print (x)
    gives you: [1, 2, 3, [4, 5]]
    ```

    * extend: Extends list by appending elements from the iterable.

    ```python
    x = [1, 2, 3]
    x.extend([4, 5])print (x)
    gives you: [1, 2, 3, 4, 5]
    ```

* iterate a list with index

    ```python
    a = [3,4,5,6]
    for i, val in enumerate(a):
        print i, val
    ```

* iterate through two lists in parallel

    ```python
    for f, b in zip(foo, bar):
    print(f, b)
    ```

* combine two list into a dict

    ```python
    dict(zip([1,2,3,4], [a,b,c,d]))
    ```

* iterate a dict

    ```python
    for (k,v) in  dict.items(): 
        print "dict[%s]=" % k,v 
    for k,v in dict.iteritems(): 
        print "dict[%s]=" % k,v 
    ```

# file operation

## file read & write

* Read binary file 

    ```python
	binfile=open(filepath,'rb')
	int(f.read(size).encode('hex'), 16)
    ```

* Read text file(Loading all lines and iterate)
    `for line in fh.readlines(): `

## directory read

* get Relative and Absolute paths of all files
    
    ```python
    import os
    def absoluteFilePaths(directory):
        for dirpath,_,filenames in os.walk(directory):
            for f in filenames:
                yield os.path.abspath(os.path.join(dirpath, f))
    ```

# advanced

* decorator(装饰器)
    * 使用Decorator

    ```python
    @dec(params)
    def method(args):
        pass
    ```

    * 不适用

    ```python
    def method(args):
        pass
    method = dec(params)(method)
    ```

* inheritance(继承)
    * `class DerivedClassName(BaseClassName):`
    * `class DerivedClassName(Base1, Base2, Base3):`

* `__slots__`
    * 功能：类似于内置的dict
    * 优势：
        * 内存使用少。和__dict__进行比较，slot和dict均为类成员，但dict由于使用的是hash的实现，更占内存
        * 访问速度快。
    * 访问模式
        * dict: `a.x --> a.__dict__ --> a.__dict__['x']`
		* slot: `b.x --> member decriptor`


# others

## shell & python

* call shell cmd in python
    * `subprocess.check_output` to do the work and get the output to python variable
    * `shell=True (using #/bin/sh)`

## draw

* 教程
    * [What is the difference between drawing plots using plot, axes or figure in matplotlib?](https://stackoverflow.com/questions/37970424/what-is-the-difference-between-drawing-plots-using-plot-axes-or-figure-in-matpl)
    * [Python Plotting With Matplotlib (Guide)](https://realpython.com/python-matplotlib-guide/)

## speedup

* Convert py to pyc

    ```python
    import py_compile
    py_compile.compile('/path/to/foo.py')
    ```

* Convert py to pyc in batch

    ```python
    import compileall
    compileall.compile_dir(r'/path')
    ```