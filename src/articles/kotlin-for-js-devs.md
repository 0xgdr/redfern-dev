---
title: "Kotlin for JS/TS Developers"
description: "A practical guide to Kotlin for JavaScript and TypeScript developers, covering lambdas, null safety, classes, coroutines, and common gotchas."
tags: ["kotlin", "javascript", "typescript", "tutorial"]
pubDate: "2026-02-13T10:00:00Z"
---

# Kotlin for JS/TS Developers

## What Is a Lambda?

A lambda is just an anonymous function — a function without a name that you can pass around as a value.

You already use them constantly in JS:

```javascript
// JS arrow functions — these are all lambdas
const double = (x) => x * 2
const greet = (name) => `Hello ${name}`
[1, 2, 3].filter((x) => x > 1)
```

In Kotlin, lambdas use curly braces instead of `=>`:

```kotlin
// Kotlin lambdas
val double = { x: Int -> x * 2 }
val greet = { name: String -> "Hello $name" }
listOf(1, 2, 3).filter { x -> x > 1 }
```

The `->` separates parameters from the body (like `=>` in JS).

### The `it` Shorthand

When a lambda has a single parameter, Kotlin lets you skip naming it and use `it`:

```kotlin
// these are identical
listOf(1, 2, 3).filter { x -> x > 1 }
listOf(1, 2, 3).filter { it > 1 }
```

There's no JS equivalent — you always have to name your parameter in arrow functions.

### Trailing Lambda Syntax

If the last parameter of a function is a lambda, you can move it outside the parentheses:

```kotlin
// these are identical
items.filter({ it > 5 })
items.filter { it > 5 }     // trailing lambda — preferred style
```

This is why Jetpack Compose UI code looks so clean:

```kotlin
Button(onClick = { doSomething() }) {
    Text("Click me")   // this trailing lambda is the button's content/children
}
```

Think of it like JSX children — the trailing lambda is what goes "inside" the component.

---

## Variables

| JS/TS | Kotlin | Notes |
|---|---|---|
| `const x = 5` | `val x = 5` | Immutable reference |
| `let x = 5` | `var x = 5` | Mutable |
| `const x: number = 5` | `val x: Int = 5` | Explicit type |

```typescript
// TypeScript
const name: string = "hello"
let count: number = 0
const items: string[] = ["a", "b"]
```

```kotlin
// Kotlin
val name: String = "hello"
var count: Int = 0
val items: List<String> = listOf("a", "b")
```

---

## Null Safety

| JS/TS | Kotlin | Notes |
|---|---|---|
| `x?.property` | `x?.property` | Safe call (identical) |
| `x ?? fallback` | `x ?: fallback` | Null coalescing / elvis |
| `x!` (TS non-null assertion) | `x!!` | Force unwrap — avoid both |
| `x as Type` | `x as Type` | Unsafe cast |
| — | `x as? Type` | Safe cast (returns null if wrong type) |

```typescript
// TypeScript
const len = name?.length
const len = name?.length ?? 0
const len = name!.length          // trust me — crashes if wrong
```

```kotlin
// Kotlin
val len = name?.length
val len = name?.length ?: 0
val len = name!!.length           // trust me — crashes if wrong
```

Key difference: In Kotlin, `String` and `String?` are different types. The compiler won't let you use a nullable where a non-null is expected.

```kotlin
val name: String = "hello"   // can NEVER be null
val name: String? = "hello"  // CAN be null
```

---

## Functions

```typescript
// TypeScript
function greet(name: string): string {
    return `Hello ${name}`
}

const greet = (name: string): string => `Hello ${name}`

function greet(name: string = "World"): string {
    return `Hello ${name}`
}
```

```kotlin
// Kotlin
fun greet(name: String): String {
    return "Hello $name"
}

// single expression shorthand
fun greet(name: String) = "Hello $name"

// default params
fun greet(name: String = "World") = "Hello $name"

// named arguments (no JS/TS equivalent)
greet(name = "Gdr")
```

---

## Lambdas — Full Comparison

```typescript
// TypeScript
const double = (x: number): number => x * 2

const add = (a: number, b: number): number => a + b

const nums = [1, 2, 3]
nums.map((x) => x * 2)
nums.filter((x) => x > 1)
nums.find((x) => x === 2)
nums.some((x) => x > 2)
nums.reduce((acc, x) => acc + x, 0)
```

```kotlin
// Kotlin
val double = { x: Int -> x * 2 }

val add = { a: Int, b: Int -> a + b }

val nums = listOf(1, 2, 3)
nums.map { it * 2 }
nums.filter { it > 1 }
nums.find { it == 2 }
nums.any { it > 2 }               // .some() in JS
nums.fold(0) { acc, x -> acc + x } // .reduce() in JS
```

### Multi-line Lambdas

```typescript
// TypeScript
const process = (items: string[]) => {
    const filtered = items.filter((x) => x.length > 3)
    const mapped = filtered.map((x) => x.toUpperCase())
    return mapped
}
```

```kotlin
// Kotlin — last expression is the return value (no `return` keyword needed)
val process = { items: List<String> ->
    val filtered = items.filter { it.length > 3 }
    val mapped = filtered.map { it.uppercase() }
    mapped  // last expression = return value
}
```

### Passing Lambdas to Functions

```typescript
// TypeScript
function doTwice(action: () => void) {
    action()
    action()
}
doTwice(() => console.log("hi"))
```

```kotlin
// Kotlin
fun doTwice(action: () -> Unit) {  // Unit = void
    action()
    action()
}
doTwice { println("hi") }         // trailing lambda
```

---

## Classes

```typescript
// TypeScript
class User {
    constructor(
        public readonly name: string,
        public readonly age: number
    ) {}
}
```

```kotlin
// Kotlin — constructor params ARE the properties
class User(val name: String, val age: Int)
```

### Data Classes (like TS interfaces + spread)

```typescript
// TypeScript
interface Todo {
    id: number
    title: string
    completed: boolean
}

const todo: Todo = { id: 1, title: "Learn Kotlin", completed: false }
const done = { ...todo, completed: true }  // immutable update
```

```kotlin
// Kotlin
data class Todo(
    val id: Int,
    val title: String,
    val completed: Boolean = false
)

val todo = Todo(id = 1, title = "Learn Kotlin")
val done = todo.copy(completed = true)  // immutable update
```

`data class` gives you for free: `equals()`, `hashCode()`, `toString()`, `copy()`.

---

## Sealed Classes (Discriminated Unions)

```typescript
// TypeScript discriminated union
type UIState =
    | { status: "loading" }
    | { status: "success"; data: User[] }
    | { status: "error"; message: string }

function render(state: UIState) {
    switch (state.status) {
        case "loading":
            return showSpinner()
        case "success":
            return showUsers(state.data)
        case "error":
            return showError(state.message)
    }
}
```

```kotlin
// Kotlin sealed class
sealed class UIState {
    object Loading : UIState()
    data class Success(val data: List<User>) : UIState()
    data class Error(val message: String) : UIState()
}

fun render(state: UIState) {
    when (state) {
        is UIState.Loading -> showSpinner()
        is UIState.Success -> showUsers(state.data)  // auto smart-cast
        is UIState.Error -> showError(state.message)  // auto smart-cast
        // no else needed — compiler knows all cases are covered
    }
}
```

Why sealed classes are better than TS unions:

- **Compiler-enforced exhaustiveness** — forget a case and it won't compile
- **Smart casting** — inside each branch the type is automatically narrowed
- **Add a new variant** and every `when` block becomes a compile error until handled

---

## Gotchas Coming from JS/TS

### No Implicit Coercion

```kotlin
val x: Int = 5
val y: Long = x          // won't compile
val y: Long = x.toLong() // must be explicit
```

### `==` Is Safe in Kotlin

```typescript
// TS
"abc" === "abc"  // strict equality (what you want)
"abc" == "abc"   // loose equality (avoid)
```

```kotlin
// Kotlin — opposite convention!
"abc" == "abc"   // structural equality (what you want)
"abc" === "abc"  // referential equality (rarely needed)
```

### No Truthiness

```typescript
// TS
if (myList.length) { ... }             // truthy check
const name = user.name || "default"    // falsy fallback
```

```kotlin
// Kotlin — must be explicit
if (myList.isNotEmpty()) { ... }
val name = user.name ?: "default"  // only null triggers this, not "" or 0
```

### String Templates

```typescript
// TS
const msg = `Hello ${name}, you are ${age} years old`
```

```kotlin
// Kotlin — double quotes, $ prefix
val msg = "Hello $name, you are $age years old"
val msg = "Length: ${name.length}"  // use braces for expressions
```

---

## Async: Coroutines vs Promises

```typescript
// TypeScript
async function fetchUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`)
    return response.json()
}
```

```kotlin
// Kotlin
suspend fun fetchUser(id: String): User {   // suspend = async
    val response = client.get("/api/users/$id")  // suspends here = await
    return response.body()
}
```

| JS/TS | Kotlin |
|---|---|
| `async function` | `suspend fun` |
| `await` | automatic at suspend points |
| `Promise<T>` | return type is just `T` |
| `Promise.all()` | `coroutineScope { async {} + async {} }` |

Coroutines need a **scope** to run in (no top-level `await`):

```kotlin
viewModelScope.launch {
    val user = fetchUser("123")  // suspends, doesn't block
    _state.value = user
}
```

---

## Quick Reference

| Concept | JS/TS | Kotlin |
|---|---|---|
| Immutable variable | `const` | `val` |
| Mutable variable | `let` | `var` |
| Lambda | `(x) => x * 2` | `{ x -> x * 2 }` or `{ it * 2 }` |
| Optional chaining | `?.` | `?.` |
| Nullish coalescing | `??` | `?:` |
| Non-null assertion | `!` | `!!` |
| String template | `` `hi ${name}` `` | `"hi $name"` |
| Spread/copy | `{ ...obj, key: val }` | `obj.copy(key = val)` |
| Void return | `void` | `Unit` |
| Discriminated union | `type X = A \| B` | `sealed class X` |
| Async function | `async function` | `suspend fun` |
| Await | `await expr` | implicit at suspend calls |
| Console log | `console.log()` | `println()` |
| Array type | `string[]` / `Array<string>` | `List<String>` |
| Object/Map type | `Record<string, number>` | `Map<String, Int>` |
