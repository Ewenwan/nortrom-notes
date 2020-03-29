
[toc]

[programming](./prog.md)

# C

* const
    * `char * const cp`: 定义一个指向字符的指针常数，即const指针
    * `const char* p`: 定义一个指向字符常数的指针
    * `char const* p`: 等同于const char* p
* 传递引用和指针的区别
    * 引用只能在定义时被初始化一次，之后不可变；指针可变；
    * 引用没有const，指针有const，const的指针不可变；
    * 引用不能为空，指针可以为空；


# semantic

* inline
    * 关键字inline 必须与函数定义体放在一起才能使函数成为内联，仅将inline 放在函数声明前面不起任何作用。
    * 定义在类声明之中的成员函数将自动地成为内联函数
        
        ```c++
        class A
        {
            public:void Foo(int x, int y) {  } // 自动地成为内联函数
        }
        ```

* 重写与重载
    * override重写 参数相同 子类与父类
    * overload重载 参数不同 类内部
    * protected可以被继承，不可被访问；virtual属性不可改变，子类的虚函数可以不用申明virtual
    * 默认为private继承（c++多重继承中默认为private继承方式 ）
    * 派生类屏蔽基类同名函数，如要调用基类的，则需要使用作用域操作符

* 异常
    * 析构函数不能抛出异常
        * 1）如果析构函数抛出异常，则异常点之后的程序不会执行，如果析构函数在异常点之后执行了某些必要的动作比如释放某些资源，则这些动作不会执行，会造成诸如资源泄漏的问题。
        * 2）通常异常发生时，c++的机制会调用已经构造对象的析构函数来释放资源，此时若析构函数本身也抛出异常，则前一个异常尚未处理，又有新的异常，会造成程序崩溃的问题。
    * 参考
        * [C++的异常处理](https://blog.csdn.net/daheiantian/article/details/6530318)

# STL

## STL behavior

* [stl do not destruct a pointer element](https://stackoverflow.com/questions/4260464/does-stdlistremove-method-call-destructor-of-each-removed-element)
    * this pointer may be a static object, deleting one of those yields undefined behavior
    * no way to figure out whether the pointee has already been released, deleting twice yields undefined behavior
    * Uninitialized pointers, also undefined behavior.
* [reason of iterator failed](https://blog.csdn.net/y1196645376/article/details/52938474)
    * 容器的迭代器为什么会失效？--> 容器的元素在容器内部搬家了。（导致门牌号失效）

    ![iterator_failed.PNG](./cpp/iterator_failed.PNG)

    * 容器元素的引用(指针)为什么会失效？-->内存发生变化
    
    ![iterator_ref_failed.PNG](./cpp/iterator_ref_failed.PNG)

# smart pointer

* `std::unique_ptr`
    * purpose: tie the lifetime of the object to a particular block of code
    * feature: automatic destruction & exception handling
* `std::shared_ptr`
    * purpose: the lifetime of your object is much more complicated, and is not tied directly to a particular section of code or to another object.
    * feature: reference counting the pointer. This does allow the pointer to be copied. When the last "reference" to the object is destroyed, the object is deleted.
* `std::weak_ptr`
    * purpose: fix dangling reference and circular references in `std::shared_ptr`
    * feature: define a weak (uncounted) reference to a shared_ptr
* 参考
    * [What is a smart pointer and when should I use one?](http://stackoverflow.com/questions/106508/what-is-a-smart-pointer-and-when-should-i-use-one)