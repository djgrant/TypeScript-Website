# TypeScript TwoSlash

A markup format for TypeScript code, ideal for creating self-contained code samples which let the TypeScript compiler do the extra leg-work. Inspired
by the [fourslash test system](https://github.com/orta/typescript-notes/blob/master/systems/testing/fourslash.md).

Used as a pre-parser before showing code samples inside the TypeScript website and to create a standard way for us
to create examples for bugs on the compiler's issue tracker.

You can preview twoslash on the TypeScript website here: https://typescriptlang.org/dev/twoslash

### What is Twoslash?

It might be easier to show instead of telling, here is an example of code from the TypeScript handbook. We'll use
twoslash to let the compiler handle error messaging and provide rich highlight info.

##### Before

> Tuple types allow you to express an array with a fixed number of elements whose types are known, but need not be the same. For example, you may want to represent a value as a pair of a `string` and a `number`:

<pre>```ts 
// Declare a tuple type
let x: [string, number];

// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
```</pre>

> When accessing an element with a known index, the correct type is retrieved:

<pre>```ts
console.log(x[0].substring(1)) // OK
console.log(x[1].substring(1)) // Error, 'number' does not have 'substring'
```</pre>

##### After

> Tuple types allow you to express an array with a fixed number of elements whose types are known, but need not be the same. For example, you may want to represent a value as a pair of a `string` and a `number`:

<pre>```ts twoslash
// @errors: 2322
// Declare a tuple type
let x: [string, number];

// Initialize it
x = ["hello", 10];
// Initialize it incorrectly
x = [10, "hello"];
```</pre>

> When accessing an element with a known index, the correct type is retrieved:

<pre>```ts
// @errors: 2339
let x: [string, number];
x = ["hello", 10]; // OK
/// ---cut---
console.log(x[0].substring(1));
console.log(x[1].substring(1));
```</pre>

[See it in action on the site](https://www.typescriptlang.org/docs/handbook/basic-types.html#tuple).

##### What Changed?

Switching this code sample to use twoslash has a few upsides:

- The error messages in both are provided by the TypeScript compiler, and so we don't need to write either "OK" or "Error"
- We explicitly mark what errors are expected in the code sample, if it doesn't occur twoslash throws
- The second example is a complete example to the compiler. This makes it available to do identifier lookups and real compiler errors, but the user only sees the last two lines.

On the flip side, it's a bit more verbose because every twoslash example is a unique compiler environment - so you need to include all the dependent code in each example.

### Features

The Twoslash markup language helps with:

- Enforcing accurate errors from a TypeScript code sample, and leaving the messaging to the compiler
- Splitting a code sample to hide distracting code
- Declaratively highlighting symbols in your code sample
- Replacing code with the results of transpilation to different files, or ancillary files like .d.ts or .map files
- Handle multi-file imports in a single code sample
- Creating a playground link for the code

### Notes

- Lines which have `// prettier-ignore` are stripped

### API

<!-- AUTO-GENERATED-CONTENT:START (FIXTURES) -->

The twoslash markup API lives inside your code samples code as comments, which can do special commands. There are the following commands:

```ts
/** Available inline flags which are not compiler flags */
export interface ExampleOptions {
  /** Let's the sample suppress all error diagnostics */
  noErrors: false
  /** An array of TS error codes, which you write as space separated - this is so the tool can know about unexpected errors */
  errors: number[]
  /** Shows the JS equivalent of the TypeScript code instead */
  showEmit: false
  /**
   * When mixed with showEmit, lets you choose the file to present instead of the source - defaults to index.js which
   * means when you just use `showEmit` above it shows the transpiled JS.
   */
  showEmittedFile: string
  /** Whether to disable the pre-cache of LSP calls for interesting identifiers, defaults to false */
  noStaticSemanticInfo: boolean
  /** Declare that the TypeScript program should edit the fsMap which is passed in, this is only useful for tool-makers, defaults to false */
  emit: boolean
  /** Declare that you don't need to validate that errors have corresponding annotations, defaults to false */
  noErrorValidation: boolean
}
```

In addition to this set, you can use `@filename` which allow for exporting between files.

Finally you can set any tsconfig compiler flag using this syntax, which you can see in some of the examples below.

### Examples

#### `compiler_errors.ts`

```ts
// @target: ES2015
// @errors: 7006

function fn(s) {
  console.log(s.subtr(3))
}

fn(42)
```

Turns to:

> ```ts
> function fn(s) {
>   console.log(s.subtr(3))
> }
>
> fn(42)
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [],
>   "staticQuickInfos": "[ 7 items ]",
>   "errors": [
>     {
>       "category": 1,
>       "code": 7006,
>       "length": 1,
>       "start": 13,
>       "line": 1,
>       "character": 12,
>       "renderedMessage": "Parameter 's' implicitly has an 'any' type.",
>       "id": "err-7006-13-1"
>     }
>   ],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/PTAEAEBcEMCcHMCmkBcoCiBlATABgIwCsAUCBIrLAPawDOaA7LrgGzHEBmArgHYDGkAJZUeoDjwAUtAJSgA3sVCg+I2lQA2iAHTqq8KVtpcARpFgSAzNOnEAvu3ESALNhtA"
> }
> ```

#### `compiler_flags.ts`

```ts
// @noImplicitAny: false
// @target: ES2015

// This will not throw because of the noImplicitAny
function fn(s) {
  console.log(s.subtr(3))
}

fn(42)
```

Turns to:

> ```ts
> // This will not throw because of the noImplicitAny
> function fn(s) {
>   console.log(s.subtr(3))
> }
>
> fn(42)
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [],
>   "staticQuickInfos": "[ 7 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/PTAEAEDsHsEkFsAOAbAlgY1QFwIKQJ4BcoAZgIbIDOApgFAgRZkBOA5tVsQKIDKATAAYAjAFZa9MABUAFqkqgA7qmTJQMLKCzTm0BaABG1dGQCuNUNBKbp1NXCRpMuArRInI6LKmiRSkABSUAJSgAN60oKDoPpTQyNQAdMjQrIEJlCb6WMz+AMxBQbQAvuIkAQAsfEEA3LRAA"
> }
> ```

#### `cuts_out_unneccessary_code.ts`

```ts
interface IdLabel {
  id: number /* some fields */
}
interface NameLabel {
  name: string /* other fields */
}
type NameOrId<T extends number | string> = T extends number ? IdLabel : NameLabel
// This comment should not be included

// ---cut---
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
  throw "unimplemented"
}

let a = createLabel("typescript")
//  ^?

let b = createLabel(2.8)
//  ^?

let c = createLabel(Math.random() ? "hello" : 42)
//  ^?
```

Turns to:

> ```ts
> function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
>   throw "unimplemented"
> }
>
> let a = createLabel("typescript")
>
> let b = createLabel(2.8)
>
> let c = createLabel(Math.random() ? "hello" : 42)
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [
>     {
>       "docs": "",
>       "kind": "query",
>       "start": 354,
>       "length": 16,
>       "text": "let a: NameLabel",
>       "offset": 4,
>       "line": 4
>     },
>     {
>       "docs": "",
>       "kind": "query",
>       "start": 390,
>       "length": 14,
>       "text": "let b: IdLabel",
>       "offset": 4,
>       "line": 6
>     },
>     {
>       "docs": "",
>       "kind": "query",
>       "start": 417,
>       "length": 26,
>       "text": "let c: IdLabel | NameLabel",
>       "offset": 4,
>       "line": 8
>     }
>   ],
>   "staticQuickInfos": "[ 14 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/JYOwLgpgTgZghgYwgAgJIBMAycBGEA2yA3ssOgFzIgCuAtnlADTID0AVMgM4D2tKMwAuk7I2LZAF8AUKEixEKAHJw+2PIRIgVESpzBRQAc2btk3MAAtoyAUJFjJUsAE8ADku0B5KBgA8AFWQIAA9IEGEqOgZkAB8ufSMAPmQAXmRAkLCImnprAH40LFwCZEplVWL8AG4pFnF-C2ARBF4+cC4Lbmp8dCpzZDxSEAR8anQIdCla8QBaOYRqMDmZqRhqYbBgbhBkBCgIOEg1AgCg0IhwkRzouL0DEENEgAoyb3KddIBKMq8fdADkkQpMgQchLFBuAB3ZAAInWwFornwEDakHQMKk0ikyLAyDgqV2+0OEGO+CeMJc7k4e2ArjAMM+NTqIIAenkpjiBgS9gcjpUngAmAB0AA5GdNWezsRBcQhuUS+eongBZQ4WIVQODhXhPT7IAowqz4fDcGGlZAAFgF4uZyDZUiAA"
> }
> ```

#### `declarations.ts`

```ts
// @declaration: true
// @showEmit
// @showEmittedFile: index.d.ts

/**
 * Gets the length of a string
 * @param value a string
 */
export function getStringLength(value: string) {
  return value.length
}
```

Turns to:

> ```ts
> /**
>  * Gets the length of a string
>  * @param value a string
>  */
> export declare function getStringLength(value: string): number
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [],
>   "staticQuickInfos": "[ 0 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/PTAEAEBMFMGMBsCGAnRAXAlgewHYC5Q1kBXaAKBAgGcALLAdwFEBbDNCscWhlttaSADEM8aAQw4YADwB0kGWipkKAKhVlQK0AHFoiwjWihROAOZoaoLADNQiUFSITTGreAAOKRM1AA3RPCkdg5OZq7AZNBS7ljIaKDWxDiwmLigpnoAyqGmADLQZhYAFP6BYiHIzgCUoADeGqDIesTIOH4BpDIm5jRkAL5kQA"
> }
> ```

#### `highlighting.ts`

```ts
function greet(person: string, date: Date) {
  console.log(`Hello ${person}, today is ${date.toDateString()}!`)
}

greet("Maddison", new Date())
//                ^^^^^^^^^^
```

Turns to:

> ```ts
> function greet(person: string, date: Date) {
>   console.log(`Hello ${person}, today is ${date.toDateString()}!`)
> }
>
> greet("Maddison", new Date())
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [
>     {
>       "kind": "highlight",
>       "position": 134,
>       "length": 10,
>       "description": "",
>       "line": 5
>     }
>   ],
>   "queries": [],
>   "staticQuickInfos": "[ 11 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/GYVwdgxgLglg9mABAcwE4FN1QBQAd2oDOCAXIoVKjGMgDSIAmAhlOmQCIvoCUiA3gChEiCAmIAbdADpxcZNgAGACXTjZiACR98RBAF96UOMwCeiGIU19mrKUc6sAypWrzuegIQLuAbgF6BATRMHAAiAFkmBgYLBFD6MHQAd0QHdGxuXwEAemzhfILC4QA9UrLygSA"
> }
> ```

#### `import_files.ts`

```ts
// @filename: file-with-export.ts
export const helloWorld = "Example string"

// @filename: index.ts
import { helloWorld } from "./file-with-export"
console.log(helloWorld)
```

Turns to:

> ```ts
> // @filename: file-with-export.ts
> export const helloWorld = "Example string"
>
> // @filename: index.ts
> import { helloWorld } from "./file-with-export"
> console.log(helloWorld)
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [],
>   "staticQuickInfos": "[ 5 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/PTAEAEDMEsBsFMB2BDAtvAXKGCC0B3aAFwAtd4APABwHsAnIgOiIGcAoS2h0AYxsRZFQJeLFg0A6vVgATUAF5QAIgCiFNFQShBdaIgDmSgNxs2ICDiRpMoPTMrN20VFyEBvEWMnSZAX2x0NKjKjMCWBMRknPRESmx8AjQIjOL6ABSe4lJ0sgCUbEA"
> }
> ```

#### `query.ts`

```ts
let foo = "hello there!"
//  ^?
```

Turns to:

> ```ts
> let foo = "hello there!"
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "ts",
>   "highlights": [],
>   "queries": [
>     {
>       "docs": "",
>       "kind": "query",
>       "start": 4,
>       "length": 15,
>       "text": "let foo: string",
>       "offset": 4,
>       "line": 0
>     }
>   ],
>   "staticQuickInfos": "[ 1 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/DYUwLgBAZg9jEF4ICIAWJjHmdAnEAhMgNwBQA9ORBAHoD8pQA"
> }
> ```

#### `showEmit.ts`

```ts
// @showEmit
// @target: ES5
// @downleveliteration

// --importHelpers on: Spread helper will be imported from 'tslib'

export function fn(arr: number[]) {
  const arr2 = [1, ...arr]
}
```

Turns to:

> ```js
> // --importHelpers on: Spread helper will be imported from 'tslib'
> var __read =
>   (this && this.__read) ||
>   function (o, n) {
>     var m = typeof Symbol === "function" && o[Symbol.iterator]
>     if (!m) return o
>     var i = m.call(o),
>       r,
>       ar = [],
>       e
>     try {
>       while ((n === void 0 || n-- > 0) && !(r = i.next()).done) ar.push(r.value)
>     } catch (error) {
>       e = { error: error }
>     } finally {
>       try {
>         if (r && !r.done && (m = i["return"])) m.call(i)
>       } finally {
>         if (e) throw e.error
>       }
>     }
>     return ar
>   }
> var __spread =
>   (this && this.__spread) ||
>   function () {
>     for (var ar = [], i = 0; i < arguments.length; i++) ar = ar.concat(__read(arguments[i]))
>     return ar
>   }
> export function fn(arr) {
>   var arr2 = __spread([1], arr)
> }
> ```

> With:

> ```json
> {
>   "code": "See above",
>   "extension": "js",
>   "highlights": [],
>   "queries": [],
>   "staticQuickInfos": "[ 0 items ]",
>   "errors": [],
>   "playgroundURL": "https://www.typescriptlang.org/play/#code/PTAEAEGcAsHsHcCiBbAlgFwFAgughgE4DmApugFyiIDKArNmOACYIB2ANiQG4nsYkE86VLFaYGoALSTUyAA6wC6ABK85AyKFGVqcgiTxNQ0NQNDxU7dqABGJULIVKSRgGYFYyUAHJ0kPjbe4iQAHk7ooK4ArqwAxsKikawAFIQElKxRyHYEANoAugCUoADemKCgsaKQEWkATKAAvKC5AIwANKAAdD1p+ZgAvphAA"
> }
> ```

### API

The API is one main exported function:

```ts
/**
 * Runs the checker against a TypeScript/JavaScript code sample returning potentially
 * difference code, and a set of annotations around how it works.
 *
 * @param code The twoslash markup'd code
 * @param extension For example: "ts", "tsx", "typescript", "javascript" or "js".
 * @param defaultOptions Allows setting any of the handbook options from outside the function, useful if you don't want LSP identifiers
 * @param tsModule An optional copy of the TypeScript import, if missing it will be require'd.
 * @param lzstringModule An optional copy of the lz-string import, if missing it will be require'd.
 * @param fsMap An optional Map object which is passed into @typescript/vfs - if you are using twoslash on the
 *              web then you'll need this to set up your lib *.d.ts files. If missing, it will use your fs.
 */
export function twoslasher(
  code: string,
  extension: string,
  defaultOptions?: Partial<ExampleOptions>,
  tsModule?: TS,
  lzstringModule?: LZ,
  fsMap?: Map<string, string>
): TwoSlashReturn
```

Which returns:

```ts
export interface TwoSlashReturn {
  /** The output code, could be TypeScript, but could also be a JS/JSON/d.ts */
  code: string
  /** The new extension type for the code, potentially changed if they've requested emitted results */
  extension: string
  /** Sample requests to highlight a particular part of the code */
  highlights: {
    kind: "highlight"
    position: number
    length: number
    description: string
    line: number
  }[]
  /** An array of LSP responses identifiers in the sample  */
  staticQuickInfos: {
    /** The string content of the node this represents (mainly for debugging) */
    targetString: string
    /** The base LSP response (the type) */
    text: string
    /** Attached JSDoc info */
    docs: string | undefined
    /** The index of the text in the file */
    start: number
    /** how long the identifier */
    length: number
    /** line number where this is found */
    line: number
    /** The character on the line */
    character: number
  }[]
  /** Requests to use the LSP to get info for a particular symbol in the source */
  queries: {
    kind: "query"
    /** What line is the highlighted identifier on? */
    line: number
    /** At what index in the line does the caret represent  */
    offset: number
    /** The text of the token which is highlighted */
    text: string
    /** Any attached JSDocs */
    docs: string | undefined
    /** The token start which the query indicates  */
    start: number
    /** The length of the token */
    length: number
  }[]
  /** Diagnostic error messages which came up when creating the program */
  errors: {
    renderedMessage: string
    id: string
    category: 0 | 1 | 2 | 3
    code: number
    start: number | undefined
    length: number | undefined
    line: number | undefined
    character: number | undefined
  }[]
  /** The URL for this sample in the playground */
  playgroundURL: string
}
```

<!-- AUTO-GENERATED-CONTENT:END -->

## Local Development

Below is a list of commands you will probably find useful. You can get debug logs by running with the env var of `DEBUG="*"`.

### `npm start` or `yarn start`

Runs the project in development/watch mode. Your project will be rebuilt upon changes. The library will be rebuilt if you make edits.

### `npm run build` or `yarn build`

Bundles the package to the `dist` folder. The package is optimized and bundled with Rollup into multiple formats (CommonJS, UMD, and ES Module).

### `npm test` or `yarn test`

Runs the test watcher (Jest) in an interactive mode. By default, runs tests related to files changed since the last commit.