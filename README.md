# boost-ts.utils

TypeScript Library to boost typed programming

<p align="center">
  <a href="https://github.com/ryokat3/boost-ts.utils">
    <img src="https://github.com/ryokat3/boost-ts.utils/actions/workflows/test.yml/badge.svg?branch=main" alt="test status" height="20">
  </a>
</p>

## partial

This library offers a partial function call with flexible argument binding. Of course, it's __type safe__.

```ts
import { partial, _1, _2 } from "boost-ts"

function sub (a:number, b:number):number {
    return a - b
}

// bind 2nd argument
const sub10 = partial(sub, _1, 10)        // type :: (a:number)=>number
console.log(sub10(100))                   // output is 90

// swap 1st and 2nd argument
const reverse_sub = partial(sub, _2, _1)  // type :: (a:number, b:number)=>number
console.log(reverse_sub(10, 100))         // output is 90
```

## mkobjmap

Type-safe map for object.

By using `Object.entries()` and `reduce()`, we can implement a `map`-like fnction for Typescript objects.

```ts
////////////////////////////////////////////////////////////////
/// Unexpected Case
////////////////////////////////////////////////////////////////

type Box<T> = { value: T }

function boxify<T>(t: T):Box<T> {
    return { value: t }
}

const data = {
    name: "John",
    age: 26
}

const unexpected = Object.entries(data).reduce((acc, [key, value])=>{
    return {
        ...acc,
        [key]: boxify(value)
    }
}, {})

// unexpected.name is ERROR!!
//
// Even with more typing, type will be like ...
// {
//     name: Box<number> | Box<string>
//     age: Box<number> | Box<string>
// }
```

We want the type `{ name: Box<string>, age: Box<number> }` in this case.

```ts
import { mkmapobj } from "boost-ts"

////////////////////////////////////////////////////////////////
// Expected Case
////////////////////////////////////////////////////////////////

type BoxMapType<T> = { [P in keyof T]: [T[P], Box<T[P]>] }

// To reuse 'mapobj', we can list all possible types as tuple
type BoxKeyType = [string, number, boolean, string[], number[]]

// Make 'map' type with Mapped Tuple Type, and apply
const mapobj = mkmapobj<BoxMapType<BoxKeyType>>()

// The dataBox type is `{ name: Box<string>, age: Box<number> }`
const dataBox = mapobj(data, boxify)

chai.assert.equal(dataBox.name.value, data.name)
chai.assert.equal(dataBox.age.value, data.age)
```

## bundle

Supposed we have an interface for set of file operations,

```ts
// What we have

interface FileOper {
    dirname: (config:Config) => string,
    read: (config:Config, name:string) => string
    write: (config:Config, name:string, content:string) => number
}
```

and `Config` is a singleton, then we expect such interface with curried functions.

```ts
// What we expect

interface CurriedFileOper {
    dirname: () => string,
    read: (name:string) => string
    write: (name:string, content:string) => number
}
```

In such cases, `bundle` is convenient.

```ts
import { bundle } from "boost-ts"

// 'bundle' curries bunch of functions
const curriedFileOper:CurriedFileOper = bundle(config, fileOper)
```

## mergeobj

Type-safe merge of key-value objects

```ts
const recordA = {
    personal: {
        name: "John",
        age: "26"
    }
}

const recordB = {
    personal: {
        age: 26,
        nationality: "American"
    }
}

const merged = mergeobj(recordA, recordB)
/*
  The type of 'merged' is

  {
      personal: {
          name: string,
          age: number,
          nationality: string
      }
  }

*/

```

- The API of partial function is inspired by [Boost C++ library](https://www.boost.org/)