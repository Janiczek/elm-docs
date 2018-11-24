# How to init Elm apps from JavaScript

**tl;dr** Giving modules with exposed `main` functions to Elm compiler results in exposing an `Elm.ModuleName.init(node: Element[, flags: Any])` function for each of them that when run starts the Elm app.

----

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

Depending on how you define the `main` value in your Elm app, the `init` function in JavaScript will need different arguments.

**TODO**
