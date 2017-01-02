---
layout: post
title:  项目进展 II
date:   2017-1-2
---

## State in SAFE
OOP 的核心是状态，SAFE 是如何实现这一点的呢？

+ `AbsState`
    - `AbsHeap`: `Map[Loc -> AbsObject]`
    - `AbsContext`: `Map[Loc -> AbsLexEnv]` in which `AbsLexEnv` is implemented with `(AbsDecEnvRec, AbsGlobalEnvRec, outer: AbsLoc)`, the first two `Env` are essentially `Map[String, AbsBinding]`

Consider some primitive instructions (NOTE: they are more abstract forms of `CFGNormalInst`, with expression arguments mostly presented in an already evaluated form):

1. Allocate a new object `CFGAlloc(x, proto, addr)`
    - prototype object `proto` is from getting the `prototype` field from the function object, for example, `Array.prototype`
    - `x: CFGId` is the left hand side variable to bind the location
    - `addr: Address` is the allocated address (XXX: Why it is given explicitly?)
    - The prototype object, in its RHS form, is in fact a `LocSet`. Why? Think about [the prototype chain](https://developer.mozilla.org/en/docs/Web/JavaScript/Inheritance_and_the_prototype_chain), the newly created instance *inherits* certain prototype by forming a link to it.
    - Now, we update the heap with a new location pointing to a `AbsObject.newObject(protoLocSet)`, and binding `x` to this location
2. Modify an object's state `CFGStore(obj, index, rhs)`
    - This is just `set` (`obj[index] <- rhs`), and trivially we can also know about `get`
    - consider each of the *set* of abstract strings that `index` represents, we will do store for each of them
    - Now differentiate the *set* of abstract objects that `obj` represents into no-array and array ones, and for each of them, we will do corresponding store using `propStore` or `arrayIdxStore`.

## OOP in SAFE


## A new idea
Now I am considering another dimension of design to solve the efficiency and state binding problem.



## TODO in next issue

- Recency abstraction in SAFE
