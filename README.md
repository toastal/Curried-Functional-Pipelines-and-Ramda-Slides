## Curried Functional
## Pipelines & Ramda

[![Ramda logo is laughably ugly—kinda like Iceweasel](http://ramda.jcphillipps.com/logo/ramdaFilled_200x235.png)](http://ramdajs.com/)

@toastal


* * *


I work in Boulder building a media application with Electron, React, Redux, Babel, & now Ramda.

My degree is in art – so obviously I’m someone you can trust.

Nothing special here: https://toast.al/


* * *


## Simple Problem: JS Object -> Query String Parameters

```js
const obj =
  { foo : "bar"
  , baz : true
  , qux : 3.1415
  }

objToQueryStr(obj)
//=> "foo=bar&baz=true&qux=3.1415"
```


* * *


## Junior Norwegian Tech:

```js
function objToQueryStr(obj) {
  var param = "";
  _.forEach(obj, function(value, key) {
      param += key + "=" + value + "&";
  });
  return param.slice(0, -1);
}
```


- - -


## `slice(0, -1)`... Really?

And why are we mutating `param`?


* * *


## Lodash:

```js
const objToQueryStr = (obj) =>
  _.join(_.map(_.toPairs(obj), (kvs) => _.join(kvs, "=")), "=")
```


- - -


## Lodash Attempt #2:

```js
const objToQueryStr = (obj) =>
   _.chain(obj)
     .toPairs()
     .map((kvs) => kvs.join("="))
     .join("&")
     .value()
```


* * *


## Problems with JS and these intermediary functions

```js
const objToQueryStr = (obj) => {
   killAllKittens()  // :(
   return _.chain(obj)
     .pairs()
     .map((kvs) => {
        call("Mom")  // :(
        return kvs.join("=")
      })
     .join("&")
     .value()
}
```


* * *


## How do we test this?


Well, we should test all of these functions because JavaScript allows side-effects.


* * *


## Moreover you're tied to `_()`

They’re not static functions. You must call `.value()` to use what’s inside and sometimes even in the chains you’ll have to call `.value()` because the query expression or transducer needs to be evaluated to be continue. You can’t just use these functions on normal data because you’re required to be in that object prototype.


* * *


## Or we could write it in a point-free matter.

"...function definitions do not identify the arguments (or “points”) on which they operate"


* * *


# Which requires: COMPOSITION


- - -


## What is composition?

### Look at Math:

```
f(g(x)) = f ∘ g
```


- - -


## How does this look in other languages?

```haskell
-- Haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
foo = f . g
```

```elm
-- Elm
(<<) : (b -> c) -> (a -> b) -> (a -> c)
foo = f << g
```

```fsharp
// F#
let foo = f << g
```

```clojure
  ;; Clojure
(def foo (comp f g))
  ```

```javascript
// ...JavaScript
```

- - -


## ECMAScript 2015

```js
const compose = (...fns) =>
  (initial) =>
    fns.reduceRight(
      (result, fn) => fn(result),
      initial
    )

const foo = compose(f, g)
```


* * *


## Back to lodash

```js
const objToQueryStr =
  _.flowRight(_.partial(_.join, _, "&"), _.partial(_.map, _, _.partial(_.join, _, "=")), _.toPairs)
```

Oh that’s no good… It seems our argument order is posing a problem.


* * *


## Currying

"Currying is the technique of translating the evaluation of a function that takes multiple arguments (or a tuple of arguments) into evaluating a sequence of functions, each with a single argument."


- - -


## Simplified


```js
// Not Curried
// notCurriedAdd : (Number, Number) -> Number
const notCurriedAdd = (a, b) =>
  a + b

// Curried
// add : Number -> Number -> Number
const add = (a) =>
  (b) =>
    a + b

// Ternary Curry
// add3Things : Number -> Number -> Number -> Number
const add3Things = (a) => (b) => (c) => a + b + c
```


- - -


## What’s this let you do?

```js
// add7 : Number -> Number
const add7 =
  add(7)

add7(3)
//=> 10

[0, 1, 2, 3, 4].map(add7)
//=> [7, 8, 9, 10, 11]
```

- - -


## Side Note about `Function.prototype.bind`

```js
const add3Things = (a, b, c) =>
  a + b + c

const add7And3 =
  add3Things.bind(null, 7, 3)

add7And3(2)
//=> 12
```

It’s pretty ugly …imagine if we didn't have to do this step.


* * *


## Why is this simpler?

![math functions!](http://www.math-only-math.com/images/436xNxmath-practice-test-on-function.jpg.pagespeed.ic.uohck83wf2.jpg)

Functions take *one thing* and return *one thing*--a lot of times that one thing is a function.

- - -


## And This is How Composition Looks

![math composition](http://image.tutorvista.com/cms/images/38/composition-of-functions-examples.JPG)


* * *


## Enter Ramda – Curried and Collection Comes Last

```js
const collection =
  [0, 1, 2, 3]

R.join(" ^_^ ", collection)
//=> 0 ^_^ 1 ^_^ 2 ^_^ 3

R.join(" ^_^ ")(collection)
//=> 0 ^_^ 1 ^_^ 2 ^_^ 3

const joinWithLobster =
  R.join(" (V)!_!(V) ")

joinWithLobster(collection)
//=> 0 (V)!_!(V) 1 (V)!_!(V) 2 (V)!_!(V) 3
```


- - -


## Our Query String Problem

```js
// compare lodash
const objToQueryStr =
  _.flowRight(_.partial(_.join, _, "&"), _.partial(_.map, _, _.partial(_.join, _, "=")), _.toPairs)

// to Ramda
// objToQueryStr_ : {k: v} -> String
const objToQueryStr_ =
  R.compose(R.join("&"), R.map(R.join("=")), R.toPairs)
```


- - -


## We can pipe too… because we don’t read Hebrew

```js
const {join, map, pipe, toPairs} = R

const objToQueryStr = pipe(  // {k: v}
  toPairs,                   // |> [[k, v]]
  map(join("=")),            // |> [String]
  join("&")                  // -> String
)

// Or to be concise
const objToQueryStr_ =
  pipe(toPairs, map(join("=")), join("&"))

objToQueryStr(obj)
//=> "foo=bar&baz=true&qux=3.1415"
```


- - -


## Remember the Viking Code From Earlier?

```js
// Long live Odin
function objToQueryStr(obj) {
  var param = "";
  _.forEach(obj, function(val, key) {
      param += key + "=" + val + "&";
  });
  return param.slice(0, -1);
}

// Lodash chains for days
const objToQueryStr_ = (obj) =>
  _.chain(obj).toPairs().map((kvs) => kvs.join("=")).join("&").value()

// Totally Ramdical dude
const objToQueryStr__ =
  pipe(toPairs, map(join("=")), join("&"))
```


* * *


## Fun with Pipe and Placeholders

```js
const {__, find, flip, lt, pipe, prop, propEq, propOr, propSatisfies, reject} = R

// type alias Child = { name : String, age : Int }

// belcherChildren : [Child]
const belcherChildren =
  [ { name : "Tina",   age : 13 }
  , { name : "Gene",   age : 11 }
  , { name : "Louise", age : 9  }
  ]

removeYoungerThan("Tina")
//=> [{name: "Tina", age: 13}]

removeYoungerThan("Gene")
//=> [{name: "Tina", age: 13}, {name: "Gene", age: 11}]
```


- - -


### Making the pipeline

```js
const belcherChildren =
  [{name:"Tina", age:13}, {name:"Gene", age:11}, {name:"Louise", age:9}]

const removeYoungerThan = pipe(  // String
  propEq("name"),                // |> (Child -> Bool)
  find(__, belcherChildren),     // |> Child | undefined
  propOr(0, "age"),              // |> Int
  flip(lt),                      // |> (Int -> Bool)
  propSatisfies(__, "age"),      // |> (Child -> Bool)
  reject(__, belcherChildren)    // -> [Child]
)

removeYoungerThan("Tina")
//=> [{name: "Tina", age: 13}]
removeYoungerThan("Gene")
//=> [{name: "Tina", age: 13}, {name: "Gene", age: 11}]
```


- - -


### Associativity means we can can build more pipelines

```js
const getBelcherChildByName = pipe(  // String
  propEq("name"),                    // |> (Child -> Bool)
  find(__, belcherChildren)          // -> Child | undefined
)

const isAgeLessThan = pipe(          // Child | undefined
  propOr(0, "age"),                  // |> Int
  flip(lt)                           // -> (Int -> Bool)
)

const removeYoungerThan = pipe(      // String
  getBelcherChildByName,             // |> Child | undefined
  isAgeLessThan,                     // |> (Int -> Bool)
  reject(__, belcherChildren)        // -> [Child]
)

removeYoungerThan("Tina")  //=> [{name: "Tina", age: 13}]
```


* * *


## Advantages of Pipelines + Currying

- Declarative
- Concise
- LEGOize your code
- Not locked into a Object’s prototype
- Cuts down on attempted side effects
- Testable


- - -


## Disadvantages

- Can be hard to follow (pointless programming)
- Requires a library or a lot of boilerplate


- - -


## Alternatives

- [lodash-fp](https://github.com/lodash/lodash/wiki/FP-Guide) :(
- [Ramtuary (Ramda + Ramda Fantasy + Sanctuary)](https://github.com/davidchase/ramtuary) :)
- [mori](https://swannodette.github.io/mori/) :D
- [Elm](http://elm-lang.org/) / [PureScript](http://www.purescript.org/) XD


- - -


## Learn More (For Great Good)

- [Hey Underscore, You’re Doing It Wrong (video)](https://www.youtube.com/watch?v=m3svKOdZijA)
- [Practical Functional Programming: Pick Two (video)](https://www.youtube.com/watch?v=XcS-LdEBUkE)
- [Elm Architecture Tutorial](https://github.com/evancz/elm-architecture-tutorial/)


* * *


# Thank You

Let’s be friends `@toastal`.
