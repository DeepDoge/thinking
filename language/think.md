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
because if they were types we would be able to call them with anything

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
