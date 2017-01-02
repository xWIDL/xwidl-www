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

## OO in SAFE
JS is an prototype-based OO language. This is also crucial to our modeling.

Let's consider some examples first.

`var x = new ClsX();` will be compiled to SAFE intermediate repr (using `cfgBuild` flag):

```
<>fun<>1 := @ToObject(ClsX) @ #1
<>arguments<>2 := allocArg(0) @ #2
<>proto<>3 := <>fun<>1["prototype"]
<>obj<>4 := alloc(<>proto<>3) @ #3
construct(<>fun<>1, <>obj<>4, <>arguments<>2) @ #4
```

A more readable version is

```
fClsX       := @ToObject(ClsX)
args        := allocArg(0)
protoClsX   := fClsX.prototype
obj         := alloc(protoClsX)

construct(fClsX, obj, args)
```

Assuming you understand the prototype-based OO mechanism in JS, let's investigate into the following issues directly:

- Where does `ClsX` exist? How to create it? (Since it looks global), how to load it into the default global environment?
    - `@ToObject` is an internal API call (`IC`), whose expression argument will be first forwarded to e`V`aluate and then try to coerce it to `AbsObject`
    - Since is a *function* object, we create it with `FuncModel`, which has a member called `protoModel` and a member called `construct`. We need to provide both
    - To load it, just modify the value of `BuiltInGlobal` object
- How is `prototype` created? How is it bound to the (constructor) function?
    - Prototype is also an object, but with an emphasize on OO, we will consider how can its operations manipulate the `this` state. This is done by a `SystemAddr` representing the instance of such prototype. You might wonder, if that is true, then `this` will only be able to refer to a single set of all *actual* instances, rather than differentiating them according to the construction site. Well, by recency abstraction, there is actually a trade-off here between efficiency and precision.


## A new idea
Now I am considering another dimension of design to solve the efficiency and state binding problem.

There are several problems of the full SMT based solution

1. Slow
2. The state maintenance is hairy, albeit necessary to support a high level of expressivity

However, some of the specification are *apparently* simple, which means that the state transformer underlying the specification can be directly encoded in static analyzer. In fact, by composing the abstract interpretations of operations, we can compose them without user's intervention.

Interestingly, we still need to do the *domain probing* when the expression might be too complex to invert, and need to fall back to the SMT solver when an abstract boolean evaluation can't work at all (consider `if ... then ... else`).

Anyway, this is an interesting direction worth exploring.
