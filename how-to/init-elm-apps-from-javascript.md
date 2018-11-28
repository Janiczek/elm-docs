# How to init Elm apps from JavaScript

> Written for Elm 0.19

**tl;dr** Compiling modules with exposed `main` functions results in exposing an `Elm.ModuleName.init(node: Element[, flags: Any])` JavaScript function for each of them that run the module's `main` function.

| Terms of Contents                               |
| ----------------------------------------------- |
| [#basic-example](Basic example)                 |
| [#multiple-entrypoints](Multiple entrypoints)   |
| [#arguments](Arguments)                         |
| [#return-value](Return value)                   |
| [#decoding-flags-in-elm](Decoding flags in Elm) |

----

## Basic example

Let's actually look at an example of that. This is all done with Elm 0.19.

```elm
module HtmlMain exposing (main)

import Html exposing (Html)


main : Html msg
main =
    Html.text "HtmlMain says hello!"
```

Let's compile that:

```bash
$ elm make HtmlMain.elm --output elm.js
```

The output JavaScript code then allows you to run that `main` function:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <script src="elm.js"></script>
  </head>
  <body>
    <div id="elm"></div>
    <script>
      Elm.HtmlMain.init({
        node: document.getElementById('elm')
      });
    </script>
  </body>
</html>
```

## Multiple entrypoints

Note that if you give multiple files to `elm make`, it will expose multiple init functions in the JS code:

```bash
$ elm make Foo.elm Bar.elm --output elm.js
```

```js
console.log(Elm);
// => {
//   Foo: {init: [Function]},
//   Bar: {init: [Function]}
// }
```

Those will thankfully share common code!

## Arguments

Depending on how you define the `main` value in your Elm app, the `init` function in JavaScript will require different arguments:

| `main` value                                                                                          | `node`             | `flags`            |
| ----------------------------------------------------------------------------------------------------- | ------------------ | ------------------ |
| [`Html msg`](https://package.elm-lang.org/packages/elm/html/latest/Html#Html)                         | :heavy_check_mark: |                    |
| [`Platform.worker`](https://package.elm-lang.org/packages/elm/core/latest/Platform#worker)            |                    | :heavy_check_mark: |
| [`Browser.sandbox`](https://package.elm-lang.org/packages/elm/browser/latest/Browser#sandbox)         | :heavy_check_mark: |                    |
| [`Browser.element`](https://package.elm-lang.org/packages/elm/browser/latest/Browser#element)         | :heavy_check_mark: | :heavy_check_mark: |
| [`Browser.document`](https://package.elm-lang.org/packages/elm/browser/latest/Browser#document)       |                    | :heavy_check_mark: |
| [`Browser.application`](https://package.elm-lang.org/packages/elm/browser/latest/Browser#application) |                    | :heavy_check_mark: |

* The `node` argument needs to be a HTML element, such as one obtained with `document.getElementById('id-of-your-div')`.
* The `flags` argument is up to you: whatever you put here, your Elm app receives as [`Json.Encode.Value`](https://package.elm-lang.org/packages/elm/json/latest/Json-Encode#Value) which you can then decode.

## Return value

Calling the `Elm.ModuleName.init()` function returns an object that will contain `ports` if you define some in your Elm app and actually use them (the dead code elimination is a real thing here!). For more informations, check a guide on ports.

## Decoding flags in Elm

Usual port rules apply to flags: only certain types are allowed.

| Elm type            | JS value example  | Elm translated value     |
| ------------------- | ----------------- | ------------------------ |
| `Bool`              | `true`            | `True`                   |
| `Int`               | `42`              | `42`                     |
| `Float`             | `10.5`            | `10.5`                   |
| `String`            | `"hello"`         | `"hello"`                |
| `Maybe a`           | `null`            | `Nothing`                |
|                     | `123`             | `Just 123`               |
| `List a`            | `[11,22]`         | `[11,22]`                |
| `Array a`           | `[11,22]`         | `Array.fromList [11,22]` |
| `()`                | whatever!         | `()`                     |
| tuples              | `[true, "hello"]` | `(True, "hello")`        |
| records             | `{foo: "bar"}`    | `{ foo = "bar" }`        |
| [`Json.Decode.Value`](https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#Value) | whatever! | `Json.Decode.Value` |

Your flags type is the first parameter of the [`Program` type](https://package.elm-lang.org/packages/elm/core/latest/Platform#Program).

If your Elm type and JS value doesn't match (say, you send `42` as a flag to a `String`-expecting program), the app doesn't initialize and instead throws a JavaScript exception. For this reason, people sometimes declare their flags type as `Json.Decode.Value`, which accepts any value, and decode it themselves with JSON decoders using [`Json.Decode.decodeValue`](https://package.elm-lang.org/packages/elm/json/latest/Json-Decode#decodeValue). That way, the app always initializes and they can recover from any decoding error by dealing with the Result returned by `decodeValue`.
