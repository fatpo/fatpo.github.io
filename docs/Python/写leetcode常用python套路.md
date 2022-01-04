# 1、背景
偶尔得知同事小明，竟然是 `leetcode` 大神，刷了1700多道，黑人问号。

见贤思齐，国哥也慢慢的开始用`python3`来刷题啦。

是时候总结一下python的常用套路。

写于 `2022年01月04日14:07:54`。

# 2、队列
```
>>> import collections
>>> de = collections.deque()
>>> de.appendleft(1)
>>> de.appendleft(2)
>>> de.appendleft(3)
>>> print(de)
deque([3, 2, 1])
>>> de.pop()
1
>>> de.pop()
2
>>> de.pop()
3
```

# 3、栈
```
v = list.pop()
list.append(v)
```
其实用`deque`去做stack也可：
```
>>> import collections
>>> de = collections.deque()
>>> de.append(1)
>>> de.append(2)
>>> de.append(3)
>>> print(de)
deque([1, 2, 3])
>>> de.pop()
3
>>> de.pop()
2
>>> de.pop()
1
```
不过用list可以少写2行代码：
```
import collections
de = collections.deque()
```
时间复杂度都是O(1)，但我个人认为list更为优秀，因为`deque`底层结构更为复杂，处理同样的事，list更快。

可以试着做一下leetcode的[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)，找找使用stack的感觉。


# 4、堆
写个堆排序：
```
>>> import heapq
>>> def heapsort(iterable):
...     h = []
...     for v in iterable:
...         heapq.heappush(h, v)
...     return [heapq.heappop(h) for i in range(len(h))]
...
>>> sorted_list = heapsort([3,4,1,2,5,9,8,7])
>>> print(sorted_list)
[1, 2, 3, 4, 5, 7, 8, 9]
```
核心就这几个：
```
heapq.heappush(h, v)
heapq.heappop(h)

>>> h=[1,6,2,3,9]
>>> heapq.heapify(h)
>>> h
[1, 3, 2, 6, 9]
```

# 5、python的list的时间复杂度
其实这些套路都是来自小明同学的经验之道。 

对话：
```
国哥：我感觉可以用list来模拟队列和栈啊。

小明：list怎么模拟队列？

国哥：入队：list.append(v)，出队：list.pop(0)。

小明：那你知道list.pop(0)的复杂度吗？

国哥：那我去查查。
```

我知道这个list是buildin模块的，要去看C语言实现： [跳转](https://github.com/python/cpython/blob/main/Objects/listobject.c)
```
https://github.com/python/cpython/blob/main/Objects/listobject.c
```

核心函数：
```
*[clinic input]
list.pop
    index: Py_ssize_t = -1
    /
Remove and return item at index (default last).
Raises IndexError if list is empty or index is out of range.
[clinic start generated code]*/

static PyObject *
list_pop_impl(PyListObject *self, Py_ssize_t index)
/*[clinic end generated code: output=6bd69dcb3f17eca8 input=b83675976f329e6f]*/
{
    PyObject *v;
    int status;

    if (Py_SIZE(self) == 0) {
        /* Special-case most common failure cause */
        PyErr_SetString(PyExc_IndexError, "pop from empty list");
        return NULL;
    }
    if (index < 0)
        index += Py_SIZE(self);
    if (!valid_index(index, Py_SIZE(self))) {
        PyErr_SetString(PyExc_IndexError, "pop index out of range");
        return NULL;
    }
    v = self->ob_item[index];
    if (index == Py_SIZE(self) - 1) {
        status = list_resize(self, Py_SIZE(self) - 1);
        if (status >= 0)
            return v; /* and v now owns the reference the list had */
        else
            return NULL;
    }
    Py_INCREF(v);
    status = list_ass_slice(self, index, index+1, (PyObject *)NULL);
    if (status < 0) {
        Py_DECREF(v);
        return NULL;
    }
    return v;
}
```
这里只有一句是核心函数，其他都是异常判断：
```
  status = list_ass_slice(self, index, index+1, (PyObject *)NULL);
```

这个`list_ass_slice`就是python著名的切片函数啦，网上有很多源码分析教程：
* [CPython3.6源码分析-PyListObject | Lx's Blog](https://he11olx.com/2018/07/15/1.CPython3.6源码分析/1.3.Python列表对象/)
* [Python 切片赋值源码分析](https://cloud.tencent.com/developer/article/1397239)

反正要知道，list底层是个数组，`list.pop(0)`的核心代码：
```
 if (d < 0) { /* Delete -d items */
    Py_ssize_t tail;
    tail = (Py_SIZE(a) - ihigh) * sizeof(PyObject *);
    
    // 左移 a[ihigh:]，减少空位，在list.pop(0)的场景下，它要挪动n-1个位置！所以是O(n)。
    // 从 memmove()也能看出来，底层是一个数组，否则链表删除没这么复杂。
    memmove(&item[ihigh+d], &item[ihigh], tail);
    
    // 拷贝后，尝试 resieze
    if (list_resize(a, Py_SIZE(a) + d) < 0) {
        // 失败后恢复 recycle
        memmove(&item[ihigh], &item[ihigh+d], tail);
        memcpy(&item[ilow], recycle, s);
        goto Error;
    }
    item = a->ob_item;
}
```

至此撒花完结。