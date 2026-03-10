# 3. Namespaces and Attribute Lookup

You already started this topic.

Topics:

* instance namespace
* class namespace
* attribute lookup algorithm
* `__dict__`
* attribute shadowing
* attribute overriding

Full lookup order:

```
data descriptor
instance dict
class dict
non-data descriptor
MRO
__getattr__
```
---
