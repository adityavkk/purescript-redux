# Tutorial

Apps using Redux maintain their whole state in an *object tree* inside a **single** store. To manipulate
such states one has to emit `actions` which are plain JavaScript objects containg the `type` of
the given action and optionally other properties, like `payload` etc.

To register a store we use `createStore` and give it a `reducer` and an optional `state`
as arguments.

In the example below register a new store by using two arguments `counter` (the reducer)
and `1` (the initial state)

**Hint**: *PureScript <a href="https://leanpub.com/purescript/read#leanpub-auto-curried-functions">doesn't have functions that take more than one argument</a>! I'm using JavaScript terms here
because Redux is written in JavaScript.*

```haskell
store <- (createStore counter 1)
```

A `reducer` is a function that takes a `state` and an `action` and returns a new `state`. This is
how our reducer looks like. It's written as a `lambda` function (a `callback` in JS) which is always
indicated with an `\` (it resembles the small greek lambda letter λ).

```haskell
counter ::  Int -> Action -> Int
counter = \v t -> case t.type of
                        "INCREMENT" -> v + 1
                        "DECREMENT" -> v - 1
                        _ -> v
```

Redux' <a href="http://redux.js.org/docs/basics/Actions.html" target="_blank">Actions</a> are just plain POJOs.

```haskell
{ "type" : "INCREMENT", payload: "TEST INCR" }
```

The definition is located in `src/Control/Monad/Eff/Redux/Redux.purs` together with our <a href=""http://www.purescript.org/learn/ffi/ target="_blank">FFI imports</a>.

```haskell
type Action a = {
  "type" :: String
  | a
}
[...]
foreign import createStore      :: forall a b. (a -> Action b -> a) -> a -> ReduxEff Store
foreign import subscribe        :: forall e. (Eff e Unit) -> Store -> ReduxEff Unit
foreign import dispatch         :: forall a. Action a -> Store -> ReduxEff (Action a)
foreign import getState         :: forall a. Store -> ReduxEff a
foreign import replaceReducer   :: Reducer -> Store -> ReduxEff Unit
```

We also want to be informed about any state changes. Therefore we define yet another callback called `numericListener`.

```haskell
numericListener :: forall e. Store -> Eff (reduxM :: ReduxM, console :: CONSOLE | e) Unit
numericListener = \store -> do
                     currentState <- (getState store)
                     log ("STATE: " <> (unsafeCoerce currentState))
```

We register it by using Redux' `subscribe` function.

```haskell
(subscribe (numericListener store) store)
```

**Hint**: The argument `store` is used to *curry* the `numericListener` callback.

And finally we register two event handlers to react to button clicks. Here we're using <a href="http://www.ractivejs.org/" target="_blank">RactiveJS</a> and
its <a href="http://docs.ractivejs.org/latest/proxy-events" target="_blank">proxy events</a>.

*This is not mandatory as you can use any other UI-Library or Framework instead.*

```haskell
on "increment-clicked" (onIncrementClicked ract) ract
on "decrement-clicked" (onDecrementClicked ract) ract
```

Our event handler functions have the same mechanics. Only their `action` types are different.

First, we get our **store** reference from the RactiveJS component via `get "store"`. Then we use Redux'
function `dispatch` to, well, *dispatch* a new action of type `INCREMENT`. Of course, we give it the
`store` instance too.

**Hint**: *PureScript <a href="https://leanpub.com/purescript/read#leanpub-auto-runtime-data-representation">natively supports</a> JavaScript's objects.*

That's why we can simply write **{ "type" : "INCREMENT" }** *without any extra calls or conversions. Just use the good old POJOs.* :smile:

```haskell
onIncrementClicked = \r e -> do
                             store <- (get "store" r)
                             log "DISPATCH: INCREMENT"
                             action <- (dispatch { "type" : "INCREMENT", payload: "TEST INCR" } store)
                             pure unit
```

This is how the demo app works.

Basically, it's a combination of a *predictable container* (Redux) with *real* pure functions and <a href="http://blog.jenkster.com/2015/12/which-programming-languages-are-functional.html" target="_blank">hostility towards any side-effects</a> (PureScript).
