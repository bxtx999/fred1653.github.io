---
title: Python的LRU Cache
date: 2020-12-30 00:12:53
tags: ["lru", "functools"]
categories: ["Python", "functools"]
desc: 在 Python 中的 functools 模块是应用高阶函数，即参数或（和）返回值为其他函数的函数。通常来说，此模块的功能适用于所有可调用对象。
---

## functools.lru_cache

在 Python 中的 functools 模块是应用高阶函数，即参数或（和）返回值为其他函数的函数。通常来说，此模块的功能适用于所有可调用对象。

<!-- more -->

```python
@functools.cache
def factorial(n):
    return n * factorial(n-1) if n else 1

>>> factorial(10)      # no previously cached result, makes 11 recursive calls
3628800
>>> factorial(5)       # just looks up cached value result
120
>>> factorial(12)      # makes two new recursive calls, the other 10 are cached
479001600
```

`functools.cache(user_function)` 是简单轻量级未绑定函数缓存。 有时称为 "memoize"。返回值与 lru_cache(maxsize=None) 相同，创建一个查找函数参数的字典的简单包装器。 因为它不需要移出旧值，所以比带有大小限制的 lru_cache() 更小更快。`

LRU函数的API是`@functools.lru_cache(user_function)` 和 `@functools.lru_cache(maxsize=128, typed=False)`，一个为函数提供缓存功能的装饰器，缓存`maxsize`组传入参数，在下次以相同参数调用时直接返回上一次的结果。用以节约高开销或I/O函数的调用时间。

由于使用了字典存储缓存，所以该函数的固定参数和关键字参数必须是可哈希的。不同模式的参数可能被视为不同从而产生多个缓存项，例如, `f(a=1, b=2)` 和 `f(b=2, a=1)` 因其参数顺序不同，可能会被缓存两次。

如果指定了`user_function`，它必须是一个可调用对象。 这允许 lru_cache 装饰器被直接应用于一个用户自定义函数，让 `maxsize` 保持其默认值 128:

```python
@lru_cache
def count_vowels(sentence):
    sentence = sentence.casefold()
    return sum(sentence.count(vowel) for vowel in 'aeiou')
```

- 如果`maxsize`设为`None`，LRU 特性将被禁用且缓存可无限增长。

- 如果`typed`设置为`true`，不同类型的函数参数将被分别缓存。例如， `f(3)` 和 `f(3.0)` 将被视为不同而分别缓存。

被包装的函数配有一个`cache_parameters()`函数，该函数返回一个新的`dict`用来显示`maxsize`和`typed`的值。 这只是出于显示信息的目的。 改变值没有任何效果。

为了衡量缓存的有效性以便调整`maxsize`形参，被装饰的函数带有一个`cache_info()`函数。当调用`cache_info()`函数时，返回一个具名元组，包含命中次数 hits，未命中次数 misses ，最大缓存数量 maxsize 和 当前缓存大小 currsize。在多线程环境中，命中数与未命中数是不完全准确的。

该装饰器也提供了一个用于清理/使缓存失效的函数`cache_clear()` 。

原始的未经装饰的函数可以通过 `__wrapped__` 属性访问。它可以用于检查、绕过缓存，或使用不同的缓存再次装饰原始函数。

[LRU（最久未使用算法）](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU))缓存 在最近的调用是即将到来的调用的最佳预测值时性能最好（例如，新闻服务器上最热门文章倾向于每天更改）。 缓存的大小限制可确保缓存不会在长期运行进程如网站服务器上无限制地增长。

一般来说，LRU缓存只在当你想要重用之前计算的结果时使用。因此，用它缓存具有副作用的函数、需要在每次调用时创建不同、易变的对象的函数或者诸如time（）或random（）之类的不纯函数是没有意义的。

静态 Web 内容的 LRU 缓存示例：

```python
@lru_cache(maxsize=32)
def get_pep(num):
    'Retrieve text of a Python Enhancement Proposal'
    resource = 'http://www.python.org/dev/peps/pep-%04d/' % num
    try:
        with urllib.request.urlopen(resource) as s:
            return s.read()
    except urllib.error.HTTPError:
        return 'Not Found'

>>> for n in 8, 290, 308, 320, 8, 218, 320, 279, 289, 320, 9991:
...     pep = get_pep(n)
...     print(n, len(pep))

>>> get_pep.cache_info()
CacheInfo(hits=3, misses=8, maxsize=32, currsize=8)
```

以下是使用缓存通过 动态规划 计算 斐波那契数列 的例子。

```python
@lru_cache(maxsize=None)
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

>>> [fib(n) for n in range(16)]
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610]

>>> fib.cache_info()
CacheInfo(hits=28, misses=16, maxsize=None, currsize=16)
```

## Python中的实现

### 在 CPython 中

```python
################################################################################
### LRU Cache function decorator
################################################################################

_CacheInfo = namedtuple("CacheInfo", ["hits", "misses", "maxsize", "currsize"])

class _HashedSeq(list):
    """ This class guarantees that hash() will be called no more than once
        per element.  This is important because the lru_cache() will hash
        the key multiple times on a cache miss.
    """

    __slots__ = 'hashvalue'

    def __init__(self, tup, hash=hash):
        self[:] = tup
        self.hashvalue = hash(tup)

    def __hash__(self):
        return self.hashvalue

def _make_key(args, kwds, typed,
             kwd_mark = (object(),),
             fasttypes = {int, str},
             tuple=tuple, type=type, len=len):
    """Make a cache key from optionally typed positional and keyword arguments
    The key is constructed in a way that is flat as possible rather than
    as a nested structure that would take more memory.
    If there is only a single argument and its data type is known to cache
    its hash value, then that argument is returned without a wrapper.  This
    saves space and improves lookup speed.
    """
    # All of code below relies on kwds preserving the order input by the user.
    # Formerly, we sorted() the kwds before looping.  The new way is *much*
    # faster; however, it means that f(x=1, y=2) will now be treated as a
    # distinct call from f(y=2, x=1) which will be cached separately.
    key = args
    if kwds:
        key += kwd_mark
        for item in kwds.items():
            key += item
    if typed:
        key += tuple(type(v) for v in args)
        if kwds:
            key += tuple(type(v) for v in kwds.values())
    elif len(key) == 1 and type(key[0]) in fasttypes:
        return key[0]
    return _HashedSeq(key)

def lru_cache(maxsize=128, typed=False):
    """Least-recently-used cache decorator.
    If *maxsize* is set to None, the LRU features are disabled and the cache
    can grow without bound.
    If *typed* is True, arguments of different types will be cached separately.
    For example, f(3.0) and f(3) will be treated as distinct calls with
    distinct results.
    Arguments to the cached function must be hashable.
    View the cache statistics named tuple (hits, misses, maxsize, currsize)
    with f.cache_info().  Clear the cache and statistics with f.cache_clear().
    Access the underlying function with f.__wrapped__.
    See:  http://en.wikipedia.org/wiki/Cache_replacement_policies#Least_recently_used_(LRU)
    """

    # Users should only access the lru_cache through its public API:
    #       cache_info, cache_clear, and f.__wrapped__
    # The internals of the lru_cache are encapsulated for thread safety and
    # to allow the implementation to change (including a possible C version).

    if isinstance(maxsize, int):
        # Negative maxsize is treated as 0
        if maxsize < 0:
            maxsize = 0
    elif callable(maxsize) and isinstance(typed, bool):
        # The user_function was passed in directly via the maxsize argument
        user_function, maxsize = maxsize, 128
        wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
        wrapper.cache_parameters = lambda : {'maxsize': maxsize, 'typed': typed}
        return update_wrapper(wrapper, user_function)
    elif maxsize is not None:
        raise TypeError(
            'Expected first argument to be an integer, a callable, or None')

    def decorating_function(user_function):
        wrapper = _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo)
        wrapper.cache_parameters = lambda : {'maxsize': maxsize, 'typed': typed}
        return update_wrapper(wrapper, user_function)

    return decorating_function

def _lru_cache_wrapper(user_function, maxsize, typed, _CacheInfo):
    # Constants shared by all lru cache instances:
    sentinel = object()          # unique object used to signal cache misses
    make_key = _make_key         # build a key from the function arguments
    PREV, NEXT, KEY, RESULT = 0, 1, 2, 3   # names for the link fields

    cache = {}
    hits = misses = 0
    full = False
    cache_get = cache.get    # bound method to lookup a key or return None
    cache_len = cache.__len__  # get cache size without calling len()
    lock = RLock()           # because linkedlist updates aren't threadsafe
    root = []                # root of the circular doubly linked list
    root[:] = [root, root, None, None]     # initialize by pointing to self

    if maxsize == 0:

        def wrapper(*args, **kwds):
            # No caching -- just a statistics update
            nonlocal misses
            misses += 1
            result = user_function(*args, **kwds)
            return result

    elif maxsize is None:

        def wrapper(*args, **kwds):
            # Simple caching without ordering or size limit
            nonlocal hits, misses
            key = make_key(args, kwds, typed)
            result = cache_get(key, sentinel)
            if result is not sentinel:
                hits += 1
                return result
            misses += 1
            result = user_function(*args, **kwds)
            cache[key] = result
            return result

    else:

        def wrapper(*args, **kwds):
            # Size limited caching that tracks accesses by recency
            nonlocal root, hits, misses, full
            key = make_key(args, kwds, typed)
            with lock:
                link = cache_get(key)
                if link is not None:
                    # Move the link to the front of the circular queue
                    link_prev, link_next, _key, result = link
                    link_prev[NEXT] = link_next
                    link_next[PREV] = link_prev
                    last = root[PREV]
                    last[NEXT] = root[PREV] = link
                    link[PREV] = last
                    link[NEXT] = root
                    hits += 1
                    return result
                misses += 1
            result = user_function(*args, **kwds)
            with lock:
                if key in cache:
                    # Getting here means that this same key was added to the
                    # cache while the lock was released.  Since the link
                    # update is already done, we need only return the
                    # computed result and update the count of misses.
                    pass
                elif full:
                    # Use the old root to store the new key and result.
                    oldroot = root
                    oldroot[KEY] = key
                    oldroot[RESULT] = result
                    # Empty the oldest link and make it the new root.
                    # Keep a reference to the old key and old result to
                    # prevent their ref counts from going to zero during the
                    # update. That will prevent potentially arbitrary object
                    # clean-up code (i.e. __del__) from running while we're
                    # still adjusting the links.
                    root = oldroot[NEXT]
                    oldkey = root[KEY]
                    oldresult = root[RESULT]
                    root[KEY] = root[RESULT] = None
                    # Now update the cache dictionary.
                    del cache[oldkey]
                    # Save the potentially reentrant cache[key] assignment
                    # for last, after the root and links have been put in
                    # a consistent state.
                    cache[key] = oldroot
                else:
                    # Put result in a new link at the front of the queue.
                    last = root[PREV]
                    link = [last, root, key, result]
                    last[NEXT] = root[PREV] = cache[key] = link
                    # Use the cache_len bound method instead of the len() function
                    # which could potentially be wrapped in an lru_cache itself.
                    full = (cache_len() >= maxsize)
            return result

    def cache_info():
        """Report cache statistics"""
        with lock:
            return _CacheInfo(hits, misses, maxsize, cache_len())

    def cache_clear():
        """Clear the cache and cache statistics"""
        nonlocal hits, misses, full
        with lock:
            cache.clear()
            root[:] = [root, root, None, None]
            hits = misses = 0
            full = False

    wrapper.cache_info = cache_info
    wrapper.cache_clear = cache_clear
    return wrapper

try:
    from _functools import _lru_cache_wrapper
except ImportError:
    pass
```

此外，`@cache` 函数就是`@lru_cache`参数`memoize`设置为`None`。

```python
################################################################################
### cache -- simplified access to the infinity cache
################################################################################

def cache(user_function, /):
    'Simple lightweight unbounded cache.  Sometimes called "memoize".'
    return lru_cache(maxsize=None)(user_function)
```

### methodtools中对lru_cache的修饰

methodtools 对 lru_cache进行了扩展。

```python

import functools
from wirerope import Wire, WireRope

__version__ = '0.4.2'
__all__ = 'lru_cache',


if hasattr(functools, 'lru_cache'):
    _functools_lru_cache = functools.lru_cache
else:
    try:
        import functools32
    except ImportError:
        # raise AttributeError about fallback failure
        functools.lru_cache  # install `functools32` to run on py2
    else:
        _functools_lru_cache = functools32.lru_cache


class _LruCacheWire(Wire):

    def __init__(self, rope, *args, **kwargs):
        super(_LruCacheWire, self).__init__(rope, *args, **kwargs)
        lru_args, lru_kwargs = rope._args
        wrapper = _functools_lru_cache(
            *lru_args, **lru_kwargs)(self.__func__)
        self.__call__ = wrapper
        self.cache_clear = wrapper.cache_clear
        self.cache_info = wrapper.cache_info

    def __call__(self, *args, **kwargs):
        # descriptor detection support - never called
        return self.__call__(*args, **kwargs)

    def _on_property(self):
        return self.__call__()


@functools.wraps(_functools_lru_cache)
def lru_cache(*args, **kwargs):
    return WireRope(_LruCacheWire, wraps=True, rope_args=(args, kwargs))
```

## 参考

1. https://github.com/youknowone/methodtools
2. https://github.com/python/cpython/tree/3.9
3. https://realpython.com/lru-cache-python/
4. [Wirerope](https://github.com/youknowone/wirerope）
