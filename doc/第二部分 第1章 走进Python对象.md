# 第一章  走进Python对象
                                                                            一花一世界，一叶一菩提  ---《华严经》


什么是对象：

Stephen Prata 在其著作《C++ primer》中，这样来描述对象：它是人们要进行研究的任何事物，从最简单的整数到复杂的飞机等均可看作对象，它不仅
能表示具体的事物，还能表示抽象的规则、计划或事件。因此对象具有状态和行为：
* 对象状态：一个对象用数据值来描述它的状态。
* 对象操作：用于改变对象的状态的一系列操作。

对象实现了数据和操作的结合，使数据和操作封装于对象的统一体中，并存储于计算机的内存之中。

在Python语言中， 一个整数，一个字符串等均是对象。它已经预定义了一些对象类型，如整型，字符串，链表，字典等。这些预定义的对象我们称之为内建对象（Build-in Object）。
Python也允许我们自定义对象，用于描述客观世界。然而，本书更多关注内建对象的行为。

对于一门编程语言来说， 内建对象组成了一个庞大的体系。 本书也旨在从其中抽丝拨茧，帮助读者去理清脉络。

Python对象是在堆上的结构体，而非栈上。并且使用特殊的机制确保这些内存空间在对象生命周期结束时，将内存回收。这些对象均是通过宏或函数进行访问。一般来说，对象不能被静态初始化。唯一的例外是
类型对象（如整数类型对象等），它们是静态初始化的。


## 定长对象与不定长对象
一般来说，对象可以分为两种，即：定长对象（如整型等）和不定长对象（如字符串对象）。那么Python如何在内存中组织这些对象呢？ 秘密就在object.h文件中。

## PyObject 与 PyVarObject
万物皆有本源。在object.h中，不难发现有以下两个结构体：

    typedef struct _object {
        Py_ssize_t ob_refcnt;   /* 对象的引用计数器 */
        struct _typeobject *ob_type; 
    } PyObject;

    typedef struct {
        PyObject ob_base;
        Py_ssize_t ob_size; /* Number of items in variable part */
    } PyVarObject;

其中PyVarObject是一个PyObject结构体加上一个ob_size。因此，对象的本源便是PyObject。所以说，Python中任何对象均可以使用'PyObject *'访问。

## 类型的对象
在PyObject结构体中，除了引用计数器还有一个表示类型指针：
    #define PyObject_VAR_HEAD      PyVarObject ob_base;

    typedef struct _typeobject {
        PyObject_VAR_HEAD
        const char *tp_name; /* For printing, in format "<module>.<name>" */
        Py_ssize_t tp_basicsize, tp_itemsize; /* For allocation */

        /* Methods to implement standard operations */

        destructor tp_dealloc;
        printfunc tp_print;
        getattrfunc tp_getattr;
        setattrfunc tp_setattr;
        PyAsyncMethods *tp_as_async; /* formerly known as tp_compare (Python 2)
                                        or tp_reserved (Python 3) */
        reprfunc tp_repr;

        /* Method suites for standard classes */

        PyNumberMethods *tp_as_number;
        PySequenceMethods *tp_as_sequence;
        PyMappingMethods *tp_as_mapping;
        
        // 以下省略

    } PyTypeObject;
类型也是一个对象，且是一个可变长对象。

/*
Objects are structures allocated on the heap.  Special rules apply to
the use of objects to ensure they are properly garbage-collected.
Objects are never allocated statically or on the stack; they must be
accessed through special macros and functions only.  (Type objects are
exceptions to the first rule; the standard types are represented by
statically initialized type objects, although work on type/class unification
for Python 2.2 made it possible to have heap-allocated type objects too).

An object has a 'reference count' that is increased or decreased when a
pointer to the object is copied or deleted; when the reference count
reaches zero there are no references to the object left and it can be
removed from the heap.

An object has a 'type' that determines what it represents and what kind
of data it contains.  An object's type is fixed when it is created.
Types themselves are represented as objects; an object contains a
pointer to the corresponding type object.  The type itself has a type
pointer pointing to the object representing the type 'type', which
contains a pointer to itself!).

Objects do not float around in memory; once allocated an object keeps
the same size and address.  Objects that must hold variable-size data
can contain pointers to variable-size parts of the object.  Not all
objects of the same type have the same size; but the size cannot change
after allocation.  (These restrictions are made so a reference to an
object can be simply a pointer -- moving an object would require
updating all the pointers, and changing an object's size would require
moving it if there was another object right next to it.)

Objects are always accessed through pointers of the type 'PyObject *'.
The type 'PyObject' is a structure that only contains the reference count
and the type pointer.  The actual memory allocated for an object
contains other data that can only be accessed after casting the pointer
to a pointer to a longer structure type.  This longer type must start
with the reference count and type fields; the macro PyObject_HEAD should be
used for this (to accommodate for future changes).  The implementation
of a particular object type can cast the object pointer to the proper
type and back.

A standard interface exists for objects that contain an array of items
whose size is determined when the object is allocated.
*/


/* Define pointers to support a doubly-linked list of all live heap objects. */
#define _PyObject_HEAD_EXTRA            \
    struct _object *_ob_next;           \
    struct _object *_ob_prev;

#define _PyObject_EXTRA_INIT 0, 0,


/* PyObject_HEAD defines the initial segment of every PyObject. */
#define PyObject_HEAD                   PyObject ob_base;

#define PyObject_HEAD_INIT(type)        \
    { _PyObject_EXTRA_INIT              \
    1, type },

#define PyVarObject_HEAD_INIT(type, size)       \
    { PyObject_HEAD_INIT(type) size },

/* PyObject_VAR_HEAD defines the initial segment of all variable-size
 * container objects.  These end with a declaration of an array with 1
 * element, but enough space is malloc'ed so that the array actually
 * has room for ob_size elements.  Note that ob_size is an element count,
 * not necessarily a byte count.
 */
#define PyObject_VAR_HEAD      PyVarObject ob_base;
#define Py_INVALID_SIZE (Py_ssize_t)-1

/* Nothing is actually declared to be a PyObject, but every pointer to
 * a Python object can be cast to a PyObject*.  This is inheritance built
 * by hand.  Similarly every pointer to a variable-size Python object can,
 * in addition, be cast to PyVarObject*.
 */
typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;

typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;


## 对象的创建


## 对象的行为


## 类型的类型


## 对象的引用计数

## 走进大师...
