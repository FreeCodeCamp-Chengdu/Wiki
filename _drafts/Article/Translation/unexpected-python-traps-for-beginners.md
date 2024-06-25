---
title: Bite code!
authorURL: ""
originalURL: https://www.bitecode.dev/p/unexpected-python-traps-for-beginners
translator: ""
reviewer: ""
---

Share this post

<!-- more -->

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa0f807f6-e96a-484b-91dd-ad3886cb687e_1024x1024.webp)

#### Unexpected python traps for beginners

www.bitecode.dev

Copy link

Facebook

Email

Note

Other

# Unexpected python traps for beginners

### I feel like a generator today

Feb 19, 2024

16

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa0f807f6-e96a-484b-91dd-ad3886cb687e_1024x1024.webp)

#### Unexpected python traps for beginners

www.bitecode.dev

Copy link

Facebook

Email

Note

Other

[

10

][1]

[

Share

][2]

## **Summary**

_You know what bloggers do when they feel lazy? They write list-type articles like "10 ways to foo your bar" and "Most popular libs to reticulate splines"._

_And I'm back from holidays, still in super lazy mode._

_So today we are going to see the little WTF that everybody trip on at least once._

_Python doesn't have as many as most languages, but it has a few fun stuff:_

-   _Inconspicuous string concatenation_
    
-   _When None is returned_
    
-   _Tuples are weird_
    
-   _Stop with_ `is` _already_
    
-   _Reference initialization is tricky_
    

## **Inconspicuous string concatenation**

Python will take this code

```
"very" "lazy"
```

And automatically convert it internally to:

```
"verylazy"
```

You have nothing to do for it, it happens when the file is parsed.

This is quite handy when you have long strings to break up:

```
msg = (
    "I want this to be on a single line when it prints "
    "but I want it to be broken into several lines in "
    "the code"
)
```

Unfortunately, beginners don't know about this, and when they make a list of strings (like in Django's settings.py), they may want to write something like this:

```
ALLOWED_HOSTS = [
    "localhost",
    "127.0.0.1",
    "bitecode.dev",
    "www.bitecode.dev"
]
```

But they miss a coma, and write this instead:

```
ALLOWED_HOSTS = [
    "localhost",
    "127.0.0.1",
    # woooops
    "bitecode.dev"
    "www.bitecode.dev"
]
```

This is valid Python code. It will not trigger a syntax error. But it will allow `bitecode.devwww.bitecode.dev` as a host, which will be very hard to debug, and subtle to find.

## **Never to return**

In Python, most functions and methods return a new value. If you sort numbers, you may use `sorted()`, which will return a new list:

```
>>> numbers = [4, 2, 3])
>>> numbers = sorted([4, 2, 3])
>>> print(numbers)
[2, 3, 4]
```

And you might use `list.sort()`, which sorts in place:

```
numbers = [4, 2, 3]
>>> numbers.sort()
>>> print(numbers)
[2, 3, 4]
```

Confusion arises, however, when newcomers try to use `sort()` like `sorted()`:

```
numbers = [4, 2, 3]
>>> numbers = numbers.sort()
>>> print(numbers)
None
```

Indeed, `list.sort()` modify the original list, it doesn't return anything. When a Python callable doesn't return anything, you get `None`.

And in our case, we just erased the original list reference from `numbers`, losing our data.

The reverse error also occurs:

```
>>> numbers = [4, 2, 3])
>>> sorted([4, 2, 3])
[2, 3, 4]
>>> print(numbers)
[4, 2, 3]
```

We never saved the new list, so `numbers` still contains the old unsorted one.

## **The curious case of the invisible tuple**

Tuples are weird. While an empty tuple should be created with parentheses:

```
>>> type(())
<class 'tuple'>
```

The tuple operator is not `()`, but the coma. For anything non-empty, parenthesis are optional:

```
>>> numbers = 1, 2, 3
>>> numbers == (1, 2, 3)
True
```

But a trailing comma can create a tuple of one element:

```
>>> 1,
(1,)
```

Which, as you can imagine, can create a few problems when subtle typos come into play. This is a list of strings:

```
>>> colors = [
... 'red',
... 'blue',
... 'green',
... ]
>>> colors
['red', 'blue', 'green']
```

But this is a tuple of list of strings:

```
>>> colors = [
... 'red',
... 'blue',
... 'green',
... ],
>>> colors
(['red', 'blue', 'green'],)
```

However, this is NOT a tuple of list of strings:

```
>>> colors = ([
... 'red',
... 'blue',
... 'green',
... ])
>>> colors
['red', 'blue', 'green']
```

Because there are no comas, and a tuple of at least one element must contain a coma.

Also, priorities are not the most obvious ones:

```
>>> -1, 1 == - 1, 1
(-1, False, 1)
>>> (-1, 1) == (-1, 1)
True
```

## **The dreadful** `is`

For some reason I cannot fathom, maybe a bad tutorial that is still up, there are plenty of people that use the `is` keyword for checking equality.

It's not the right tool for the job, you will almost never need `is` in all your Pythonista life, it's a very niche operator that checks identity.

Equality is checked with `==`, and this is what you want 99.999999999999999999% of the time:

```
>>> site = "bitecode.dev"
>>> site == "bitecode.dev"
True
>>> site is "bitecode.dev"
False
```

`==` checks that two different objects are functionally equivalent (e.g: two strings contain the same letters, two sets contains the same elements), while `is` checks that they are actually two references to the same objects (they are not two things, but one used in two places).

The problem is, because of the way CPython caches string, it feels like sometimes it works:

```
>>> site = "bitecode"
>>> site is "bitecode"
True
```

It seems to work here because the string is short, and is cached by the Python shell, so it's effectively the same object behind. Don't get fooled. Use `==`.

There is also an old stylistic pattern like:

```
>>> answer = None
>>> if answer is None:
...     print("The user didn't answer")
...
The user didn't answer
```

That always works because `None` is a very special object that is always cached, so `None` is always the same object everywhere in the program. I would recommend to use `==` here as well to avoid the confusion.

## **I understand that reference**

Python can multiply strings, and that's handy:

```
>>> "0" * 3
'000'
```

And also tuples:

```
>>> (0,) * 3
(0, 0, 0)
```

Very nice to initialize.

Let's do tuple of lists:

```
>>> ([0]) * 3
([0], [0], [0])
```

That seems great!

Yet it's not:

```
>>> zeros[0][0] += 1
>>> zeros
([1], [1], [1])
```

No, Python didn't magically apply the `+1` to all lists. It's just that when you do `*3`, you don't create a tuple of 3 lists. You create a tuple of 3 references, each of them to the same single list. It's the same list 3 times, but referenced in 3 difference places in the tuple. There is only one list, so when you modify it, you see it modified, and it's in 3 places, so you see it 3 times.

The safe way to do this is to use a comprehension list:

```
>>> zeros = tuple([0] for _ in range(3))
```

BTW, weird reference initialization is one of the oldest Python gotchas. You find it in mutable default values for functions:

```
>>> def a_bugged_function(reused_list=[]):
...     reused_list.append("woops")
...     return reused_list
...
>>> a_bugged_function()
['woops']
>>> a_bugged_function()
['woops', 'woops']
>>> a_bugged_function()
['woops', 'woops', 'woops']
```

`reused_list` is not a new list every time you call the function. Python creates the list once at the beginning of the program and keep the same again and again as a default value. Since we modify it, we keep adding elements to it. It's rarely what people want.

Use `None` for those:

```
>>> def a_bugged_function(reused_list=None):
...     reused_list = reused_list or []
...     reused_list.append("woops")
...     return reused_list
```

Same problem with class attributes:

```
>>> class Person:
...     friends = []
...
>>> doris = Person()
>>> julie = Person()
>>> doris.friends.append('julio')
>>> julie.friends
['julio']
```

Attributes defined this way are shared at the class level, not the instance level. So here, all objects created from the class `Person` share the same reference to the same list.

Use `__init__` or `dataclas instead`:

```
>>> class Person:
...     def __init__(self):
...          self.friends = []
```

If Python references are weird to you, [we have an article on that][3].

Try an SQL injection with your email in there. Come on bobby tables! Could be fun!

16

Share this post

![](https://substackcdn.com/image/fetch/w_120,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa0f807f6-e96a-484b-91dd-ad3886cb687e_1024x1024.webp)

#### Unexpected python traps for beginners

www.bitecode.dev

Copy link

Facebook

Email

Note

Other

[

10

][4]

[

Share

][5]

Previous

Next

[1]: https://www.bitecode.dev/p/unexpected-python-traps-for-beginners/comments
[2]: javascript:void(0)
[3]: https://www.bitecode.dev/p/python-variables-references-and-mutability
[4]: https://www.bitecode.dev/p/unexpected-python-traps-for-beginners/comments
[5]: javascript:void(0)