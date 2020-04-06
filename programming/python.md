
[toc]

[programming](./prog.md)

# app

* get Relative and Absolute paths of all files
    
    ```python
    import os
    def absoluteFilePaths(directory):
        for dirpath,_,filenames in os.walk(directory):
            for f in filenames:
                yield os.path.abspath(os.path.join(dirpath, f))
    ```

* assign value in a loop in one line

    ```python
    squares = [x**2 for x in range(10)]
    [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
    ```

# list/set/dict

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