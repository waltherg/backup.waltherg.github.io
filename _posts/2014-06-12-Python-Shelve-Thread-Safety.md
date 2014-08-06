---
layout: default
title: "Python Shelve Thread Safety"
date: 2014-06-12
tags: Python
---

# Python Shelve Thread Safety

As the documentation of `shelve` says, `shelve.Shelf` objects are not thread
safe:

> The shelve module does not support concurrent read/write access to shelved objects.
>
> --<quote>https://docs.python.org/2/library/shelve.html#restrictions</quote>

To write to a `Shelf` object in an environment where multiple threads may end
up writing to it, use a global `threading.Lock` object.

    from threading import Lock
    import shelve
    
    mutex = Lock()
    
    mutex.acquire()
    db = shelve.open(db_name)
    # write to db
    db.close()
    mutex.release()

