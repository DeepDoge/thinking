alright thinking about my imaginary language again

my previous vision was:

```ts
type NonZeroU8 = u8 & 1..255

func NonZeroU8(value: u8): NonZeroU8 | None {
    // return value // compile time error: u8 & 0..255 is not assignable to u8 & 1..255
    return value == 0 ? None : value
}

var foo = NonZeroU8(5) // type u8 & 5, foo satisfies NonZeroU8
if (foo == 0) {} // grayed out unreachable code
foo = 0 // compile time error: u8 & 0 is not assignable to u8 & 1..255

var bar: u8 = NonZeroU8(5) // type u8 & 5, bar satisfies u8
if (bar == 4) {} // grayed out unreachable code
bar = 0 // type u8 & 0, bar satisfies u8

// Don't even need NonZeroU8
func baz(non_zero_u8: u8 & 1..255) {}

var random_u8_value = random_u8() // type u8, random_u8_value satisfies u8
baz(random_u8_value) // compile time error: u8 & 0..255 is not assignable to u8 & 1..255

if (random_u8_value > 0) {
    random_u8_value // type u8 & 1..255, random_u8_value satisfies u8
    baz(random_u8_value)
}

// or
func is_non_zero(value: number) {
    return value != 0
}

if (is_non_zero(random_u8_value)) {
    random_u8_value // type u8 & 1..255, random_u8_value satisfies u8
    baz(random_u8_value)
}
```

ok example above shows how type of a value updates as the code progresses

now im gonna go more into typing system.

so we have both `type` and `trait` keywords for defining types and traits.
a trait is an interface or lets say its same as typescript types, which means there is no branding
traits define what something has so example

```ts
type Foo { foo: string }
type Foo1 { foo: string }
trait FooLike { foo: string }


var foo = Foo { foo: string }
var foo1 = Foo1 { foo: string }

func getFoo(foo: Foo) {}
getFoo(foo) // works
getFoo(foo1) // fails

func getFooLike(foo: FooLike) {}
getFooLike(foo) // works
getFooLike(foo1) // works
```

also any anonymously defined argument types are considered traits,
because if they were types we wouldn't be able to call them with anything
because remember, types are branded, and not signature based

```ts
type Foo { foo: string }
func getFooLike(foo: { foo: string }) {}

getFooLike(foo) // works
```

simple

types can't have functions in them
but you can do this.

```ts
type Foo { foo: string }

func print(foo: Foo) {
    log("foo: {foo.foo}")
}

var foo = Foo { foo: "hello" }
foo.print() // "foo: hello"
```

but traits can have functions in them

```ts
trait Stringable {
    toString(): string
}
func toString(foo: Foo) {
    return "Foo: {foo.foo}"
}
func toString(unknown: {}) { // notice how we anonymously defined an empty trait
    return "unknown value"
}

type Bar { bar: Bar }

var bar = Bar { bar: "bar" }
bar.toString() // "unknown value"
```

of course this type of typing requires namespacing instead of file based imports
which can be an issue, but i really hate the rust impl syntax, so its this or some other way

anyway btw you can do these with blocks:

```ts
var two = {
    var one = 1
    return one * 2
}

var four = (log("hello"), two * two)
```

ok some examples with "advanced" types

```ts
type Cat
type Dog
type Duck

type Pet = Cat | Dog | Duck

trait NoiseMaker { noise() }
func noise(cat: Cat) {
    log("meow")
}
func noise(dog: Dog) {
    log("woof")
}
func noise(duck: Duck) {
    log("quack")
}
func noise(other: Pet) {
    log("...")
}

func makeNoise(noiseMaker: NoiseMaker) {
    noiseMaker.noise()
}
```

you can also force all pets to implement NoiseMaker

```ts
type Cat
type Dog
type Duck

Pet satisfies NoiseMaker // if Pet doesn't satify NoiseMaker trait then this line will give compile error
type Pet = Cat | Dog | Duck

trait NoiseMaker { noise() }
func noise(cat: Cat) {
    log("meow")
}
func noise(dog: Dog) {
    log("woof")
}
func noise(duck: Duck) {
    log("quack")
}
func noise(other: Pet) {
    log("...")
}

func makeNoise(noiseMaker: NoiseMaker) {
    noiseMaker.noise()
}
```

also lets not forget the `ref`keyword
we dont want random symbols or characters making the code weird to read like in Rust or C++.

```ts
func noise(duck: ref Duck) {
    // ...
}
```

function above gets a reference to the noise, not copy.
and actually you have to use ref in order to implement trait functions.
so all of the above trait function examples should say `ref` before type

we also have `in` keyword which is same as `ref` but immutable/readonly

```ts
func noise(duck: in Duck) {
    /// ...
}
```

also we have `export` keyword, to expose stuff out in many contexts.

```ts
func example() {
    var a = 1
    export a
    var b = 2
    export b
}
```

which does the same thing as:

```ts
func example() {
    var a = 1
    var b = 2

    return { a, b } // also creates an anonymous type (not trait)
}
```

also for trait function you also have to use `export` keyword too

```ts
export func noise(_: in Duck) {
    log("quack")
}
```

so all of the trait functions above should be defined like this ^ example above

ok namespaces also use export keyword so some example

`file_a`

```ts
namespace example {
    func hello() { // private
        log("hello")
    }

    export func sayHello() { // exposed
        hello()
    }
}
```

`file_b`

```ts
example.hello(); // fails
example.sayHello(); // works

namespace example {
  sayHello(); // works
  hello(); // fails
}
```

advanced namespace behaviour

```ts
namespace example {
    export type Foo { value: string }
    export func printFoo(foo: in { value: string }) {
        log(foo.value)
    }

    var foo = { value: "hello" }
    foo.printFoo() // logs: "hello"
}

using example { Foo }

var foo = Foo { value: "hello" }
foo.printFoo(); // logs: "hello"

var bar = { value: "hello" }
bar.printFoo(); // fails
```

so we have a function called printFoo
that gets an arg that satisfies { value: string } anonymous trait

so when we define a value called `foo` inside the `example` namespace we can say `foo.printFoo()`.

but when we do the same thing outside the `example` namespace
with the anonymous type { value: string } of `bar` it fails.

its because the anonymous type of `bar` doesnt belong to `example` namespace.

but when we use the type `Foo` that is imported from the namespace `example` we can say `printFoo()` because `Foo`defined inside the
same namespace as the `printFoo()` function.

i thought about this behavior to prevent the autocomplete trashing

but if i can find a better solution instead of using namespaces
then maybe i can throw this into trash
