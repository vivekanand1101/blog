---
title: "Django's transaction.on_commit"
date: 2018-12-18
categories: 
  - "tech"
---

I am talking with Django's 1.11 in mind.

Django provides a method using which you can execute a particular function when you are sure that a particular entry has made to the database, that's transaction.on\_commit.

There is also Django's Post save signal. Both have their use cases. The difference between the two is that on\_commit is executed when Django finally commits to the database and it's only useful when you are inside a transaction. While, in case of post save signal, you are not even sure that the object you are currently saving will finally be committed to the database (if it's inside a transaction block). Post save has it's use case when you need to know the object which just got changed or got created since, you get boolean on whether it was updated or created and also the changed/created object itself.

How to work with transaction.on\_commit?

When the code is running, you can register a function to be called, by calling

_transaction.on\_commit(my\_fancy\_function\_to\_be\_called\_on\_commit)._

Remember, you are trying to pass a function here and not the function's returned value. What happens here is, Django adds the _my\_fancy\_function\_to\_be\_called\_on\_commit_ to a list of functions which it calls when it finally commits the ongoing transaction. There are few small things here:

1. Since the transactions can be nested, when does django finally execute these functions?Django executes them after it is no longer inside an atomic block (wrapped in a transaction). If there is no transaction block and you are trying to register the function, it won't get register rather get called instead. Here is the link to the source code: [Registering and What happens on registering](https://github.com/django/django/blob/stable/1.11.x/django/db/backends/base/base.py#L634) [Things happening while executing the registered functions](https://github.com/django/django/blob/stable/1.11.x/django/db/backends/base/base.py#L645)
2. How do i pass arguments to the above functions?This is actually a basic python issue. You can wrap your function which has arguments inside a lambda expression or a normal function.
    
    transaction.on\_commit(lambda: my\_fancier\_func\_with\_args(a, b))
    
     
3. From where do we get this "transaction"?By importing:
    
    ```
    from django.db import transaction
    ```
