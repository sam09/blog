---
layout: post
title: Parameterized Tests in Kotlin
published: true
---



[JUnit5](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests) introduced a shiny new feature - parameterized tests. Parameterized tests allow the user to run a test multiple times with different arguments. Parameterized tests are declared with a `@ParameterizedTest` annotation instead of the usual `@Test`. They must also provide at least one source which provides the arguments for each run.

### Getting Started

Before starting, we need to add `junit-jupiter-params` dependency in our build script. If you are using `gradle` , add the following to your dependencies

```groovy
testImplementation 'org.junit.jupiter:junit-jupiter-params:5.1.0'
```

Let's move on to writing some tests.


### Simple example

Consider the following function.


```kotlin
fun isPalindrome(candidate: String): Boolean {
    return candidate.toLowerCase() == candidate.toLowerCase().reversed()
}
```

The above function takes a string as an input and returns true if it is a palindrome, false if it is not. The string is converted to lowercase to ensure that an input like "Bob" returns true. There are a few different classes of inputs to test the above method
 
- An empty string should return true
- A palindrome like "pop" should return true.
- A palindrome with capital letters should also return true.

However, instead of writing a test for all of these scenarios, we could leverage the `@ParameterizedTest` and write a single test with multiple arguments.

```kotlin
@ParameterizedTest(name = "isPalindrome should return true for {0}")
@ValueSource(strings = ["Bob", "racecar", "Malayalam", ""])
fun `Function isPalindrome returns true for Palindromes`(candidate: String) {
    assertTrue(isPalindrome(candidate))
}
```

The above test is annotated with `ParameterizedTest`, which means that it expects a set of inputs and will run the test for each different argument. We provide the input using the `@ValueSource` annotation. The `ValueSource` is the simplest source that expects an array of literals. It can be used to provide only one argument per invocation of the test.

### Customizing The Display Name

The display name  can be customized using the `name` attribute of `@ParamterizedTest` annotation like we did in the previous example. We used `{0}` as a placeholder for the first argument in the function. Other supported placeholders are:-

- `{index}` : The current invocation index (1-based)
- `{arguments}` : The complete, comma-separated arguments list
- `{argumentsWithNames}` : The complete, comma-separated arguments list with parameter names
- `{n}` : nth argument (0-based) 


### Sources

We already saw `@ValueSource` which can be used to provide an array of literals as arguments for the tests. There are other sources as well. The full list can be found on [the Junit documentation pages](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests).

#### Method Source
The `@MethodSource` allows you to refer to a method of the test class or an *external class*. The only limitation is that this method must be static. This method can be used to provide multiple arguments.

Consider the following class which implements a complex number and also implements the addition operator for two complex numbers.

```kotlin

data class Complex(val real: Double, val img: Double) {

    operator fun plus (other: Complex): Complex {
        return Complex(this.real + other.real, this.img + other.img)
    }
}
```

Let's write a simple test to check if addition works as expected for complex numbers. Addition must satisfy the following conditions
- It should be commutative, i.e `a + b` = `b + a`
- Addition with zero should return the element itself, i.e. `a + 0 = a`

A test for the above class looks something as follows.

```kotlin

@Test
fun `verify addition`() {
    val a = Complex(1.0, 2.0)
    val b = Complex(2.0, 4.0)
    val c = Complex(3.0, 6.0)
    val zero = Complex(0.0, 0.0)

    assertEquals(a + b, c)
    assertEquals(b + a, a)
    assertEquals(a + zero, a)
}

```

However, the above is not a comprehensive test, simply because it doesn't test for a larger set of numbers. Our ideal test scenario should involve negative integers and decimal numbers. We could acheive that by modifying our test to accept a `ParameterizedTest` with a `@MethodSource`.

```kotlin
class ComplexTest {

    @ParameterizedTest(name = "{0} + {1} should {2}")
    @MethodSource("getData")
    fun `Addition works as expected`(a: Complex, b: Complex, c: Complex) {
        val zero = Complex(0.0, 0.0)
        assertEquals(a + b, c)
        assertEquals(b + a, c)
        assertEquals(a + zero, a)
    }

    companion object {
        @JvmStatic
        fun getData(): List<Arguments> {
            return (0..10).map { _ ->
                val r1 = Random.nextDouble()
                val r2 = Random.nextDouble()
                val i1 = Random.nextDouble()
                val i2 = Random.nextDouble()
                Arguments.of(
                    Complex(r1, i1), Complex(r2, i2), Complex(r1+r2, i1+i2)
                )
            }
        }
    }
```

We create a static method called `getData`, which returns a list of 10 random argument sets. Each item in the list are the three arguments for a single test run. The first two elements in each argument set are two random complex numbers and their sum is the third element. The `@MethodSource` is very valuable in cases when you want to pass a long list, or a range as an argument to the parameterized test.


#### CsvSource and CsvFileSource

Another important source is the `@CsvSource` annotation which allows you to pass a set of csv values as arguments. The default delimiter is ',' but this can be changed using the `delimiter` parameter.

The `@CsvFileSource` annotation allows you to pass a file as the source of arguments. It has a parameter `numLinesToSkip` which can be used to skip csv headers. Lines beginning with a `#` are considered as comments and not processed as arguments.


### Further Reading

The [Junit user guide](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests) has a comprehensive list of argument sources and wide variety of examples. Most of the examples are in JAVA but the underlying concepts work for kotlin too.

