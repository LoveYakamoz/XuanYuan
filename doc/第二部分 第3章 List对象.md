# List对象
序列是Python中最基本的数据结构。序列中的每个元素都分配一个数字，即它的位置 ，也称为索引，第一个索引是0，第二个索引是1，依此类推。

Python有6个序列的内置类型，但最常见的是列表和元组。
序列都可以进行的操作包括索引，切片，加，乘，检查成员。

此外，Python已经内置确定序列的长度以及确定最大和最小的元素的方法。
列表是最常用的Python数据类型，它可以作为一个方括号内的逗号分隔值出现。而且列表的数据项不需要具有相同的类型。

## 创建列表
创建一个列表，只要把逗号分隔的不同的数据项使用方括号括起来即可。如下所示：

    >>> lan_list = ['python', 'java', 'c', 'c++']
    >>> print (lan_list)
    ['python', 'java', 'c', 'c++']
    >>>



## 更新列表
我们可以对列表中某一项进行修改，也可以使用append操作增加一个列表项

    >>> lan_list[1] = 'ruby'
    >>> print (lan_list)
    ['python', 'ruby', 'c', 'c++']
    >>>



# 删除列表元素
使用del语句来删除列表中某一元素，如：

    >>> del lan_list[2]
    >>> print (lan_list)
    ['python', 'ruby', 'c++']
    >>>

从以上简单的例子，我们可以看：List是一个可以修改，增加，删除的对象，很明显，它是一个可变对象。


# List对象定义

    typedef struct {
        PyObject_VAR_HEAD
        /* Vector of pointers to list elements.  list[0] is ob_item[0], etc. */
        PyObject **ob_item;

        /* ob_item contains space for 'allocated' elements.  The number
        * currently in use is ob_size.
        * Invariants:
        *     0 <= ob_size <= allocated
        *     len(list) == ob_size
        *     ob_item == NULL implies ob_size == allocated == 0
        * list.sort() temporarily sets allocated to -1 to detect mutations.
        *
        * Items must normally not be NULL, except during construction when
        * the list is not yet visible outside the function that builds it.
        */
        Py_ssize_t allocated;
    } PyListObject;
对象的类型定义为：

    PyTypeObject PyList_Type = {
        PyVarObject_HEAD_INIT(&PyType_Type, 0)
        "list",
        sizeof(PyListObject),
        0,
        (destructor)list_dealloc,                   /* tp_dealloc */
        0,                                          /* tp_print */
        0,                                          /* tp_getattr */
        0,                                          /* tp_setattr */
        0,                                          /* tp_reserved */
        (reprfunc)list_repr,                        /* tp_repr */
        0,                                          /* tp_as_number */
        &list_as_sequence,                          /* tp_as_sequence */
        &list_as_mapping,                           /* tp_as_mapping */
        PyObject_HashNotImplemented,                /* tp_hash */
        0,                                          /* tp_call */
        0,                                          /* tp_str */
        PyObject_GenericGetAttr,                    /* tp_getattro */
        0,                                          /* tp_setattro */
        0,                                          /* tp_as_buffer */
        Py_TPFLAGS_DEFAULT | Py_TPFLAGS_HAVE_GC |
            Py_TPFLAGS_BASETYPE | Py_TPFLAGS_LIST_SUBCLASS,         /* tp_flags */
        list_doc,                                   /* tp_doc */
        (traverseproc)list_traverse,                /* tp_traverse */
        (inquiry)list_clear,                        /* tp_clear */
        list_richcompare,                           /* tp_richcompare */
        0,                                          /* tp_weaklistoffset */
        list_iter,                                  /* tp_iter */
        0,                                          /* tp_iternext */
        list_methods,                               /* tp_methods */
        0,                                          /* tp_members */
        0,                                          /* tp_getset */
        0,                                          /* tp_base */
        0,                                          /* tp_dict */
        0,                                          /* tp_descr_get */
        0,                                          /* tp_descr_set */
        0,                                          /* tp_dictoffset */
        (initproc)list_init,                        /* tp_init */
        PyType_GenericAlloc,                        /* tp_alloc */
        PyType_GenericNew,                          /* tp_new */
        PyObject_GC_Del,                            /* tp_free */
    };
其中list_methods为：

    static PyMethodDef list_methods[] = {
        {"__getitem__", (PyCFunction)list_subscript, METH_O|METH_COEXIST, getitem_doc},
        {"__reversed__",(PyCFunction)list_reversed, METH_NOARGS, reversed_doc},
        {"__sizeof__",  (PyCFunction)list_sizeof, METH_NOARGS, sizeof_doc},
        {"clear",           (PyCFunction)listclear,   METH_NOARGS, clear_doc},
        {"copy",            (PyCFunction)listcopy,   METH_NOARGS, copy_doc},
        {"append",          (PyCFunction)listappend,  METH_O, append_doc},
        {"insert",          (PyCFunction)listinsert,  METH_VARARGS, insert_doc},
        {"extend",          (PyCFunction)listextend,  METH_O, extend_doc},
        {"pop",             (PyCFunction)listpop,     METH_VARARGS, pop_doc},
        {"remove",          (PyCFunction)listremove,  METH_O, remove_doc},
        {"index",           (PyCFunction)listindex,   METH_VARARGS, index_doc},
        {"count",           (PyCFunction)listcount,   METH_O, count_doc},
        {"reverse",         (PyCFunction)listreverse, METH_NOARGS, reverse_doc},
        {"sort",            (PyCFunction)listsort,    METH_VARARGS | METH_KEYWORDS, sort_doc},
        {NULL,              NULL}           /* sentinel */
    };
其中list_as_sequence为：

    static PySequenceMethods list_as_sequence = {
        (lenfunc)list_length,                       /* sq_length */
        (binaryfunc)list_concat,                    /* sq_concat */
        (ssizeargfunc)list_repeat,                  /* sq_repeat */
        (ssizeargfunc)list_item,                    /* sq_item */
        0,                                          /* sq_slice */
        (ssizeobjargproc)list_ass_item,             /* sq_ass_item */
        0,                                          /* sq_ass_slice */
        (objobjproc)list_contains,                  /* sq_contains */
        (binaryfunc)list_inplace_concat,            /* sq_inplace_concat */
        (ssizeargfunc)list_inplace_repeat,          /* sq_inplace_repeat */
    };
            
问题: 上面提到的sq_length与下面的PyList_Size有何异同？ 是什么关系？

主要提供以下操作：

    函数声明：PyAPI_FUNC(PyObject *) PyList_New(Py_ssize_t size);
    功能描述：创建List

******
    函数声明：PyAPI_FUNC(Py_ssize_t) PyList_Size(PyObject *);
    功能描述：
******

    函数声明：PyAPI_FUNC(PyObject *) PyList_GetItem(PyObject *, Py_ssize_t);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_SetItem(PyObject *, Py_ssize_t, PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_Insert(PyObject *, Py_ssize_t, PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_Append(PyObject *, PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(PyObject *) PyList_GetSlice(PyObject *, Py_ssize_t, Py_ssize_t);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_SetSlice(PyObject *, Py_ssize_t, Py_ssize_t, PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_Sort(PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(int) PyList_Reverse(PyObject *);
    功能描述：
******
    函数声明：PyAPI_FUNC(PyObject *) PyList_AsTuple(PyObject *);
    功能描述：
******
    #ifndef Py_LIMITED_API
    函数声明：PyAPI_FUNC(PyObject *) _PyList_Extend(PyListObject *, PyObject *);

    函数声明：PyAPI_FUNC(int) PyList_ClearFreeList(void);
    函数声明：PyAPI_FUNC(void) _PyList_DebugMallocStats(FILE *out);
    #endif

    /* 以下宏定义是为了提升效率，但是少了类型的检查 */
    #ifndef Py_LIMITED_API
    #define PyList_GET_ITEM(op, i) (((PyListObject *)(op))->ob_item[i])
    #define PyList_SET_ITEM(op, i, v) (((PyListObject *)(op))->ob_item[i] = (v))
    #define PyList_GET_SIZE(op)    Py_SIZE(op)
    #define _PyList_ITEMS(op)      (((PyListObject *)(op))->ob_item)
    #endif

# 走进大师
