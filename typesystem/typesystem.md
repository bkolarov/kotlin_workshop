# The Kotlin type system

## Nullability

* Kotlin provides an environment that helps you avoid NPEs and gives errors at compile time.
* The nullability is part of its type system.

### Nullable types

* The programmer can indicate which variables or properties are allowed to be `null`.<br>
    When you attempt to call a method on a `nullable` variable, Kotlin knows that this isn't safe, therefore it will prevent possible exceptions.
    
    ```java
    int strLen(String s) {
      return s.length();
    }
    ```
    
    Guess what happens if you pass null to this function. That's right. You cry. Or you recieve a JIRA ticket and you cry. Or you see a Crashlytics report and you cry.<br>
    
    Of course you can add a `null` check in the function or annotate the parameter with `@NonNull` so the caller would know how is the function intended to be used.
    
    However, even with the annotation, you'll be able to compile your code and shoot yourself in the leg.
    
    Let's see the same functino in Kotlin.
    
    ```Kotlin
    fun strLen(s: String): Int = s.length
    ```
    Here the parameter `s` is declared as `non-null`. If you attempt to call it with `null`, you'll get a compile time error.
    
    ```
    >>> strLen(null)
    ERROR: Null can not be a value of a non-null type String
    ```
    If you want to allow null values for `s` you have to put a question mark after the type name:
    ```Kotlin
    fun strLenSafe(s: String?) = ...
    ```
    
* Put `?` after any type to mark that the variable of that type can store `null` values. `String?, Int?, AwesomeType?`.
![Nullable Type](/typesystem/nullable_type.png)

* Once you have a value of `nullable` type, the set of operations you can perform on it is restricted.
    ```
    >>> fun strLenSafe(s: String?) = s.length()
    ERROR: only safe (?.) or non-null asserted (!!.) calls are allowed
    on a nullable receiver of type kotlin.String?
    ```
* You can't assign a `nullable` variable to a `non-null` one, or you can't pass the `nullable` variable's value to a function that has a `non-null` parameter.
    ```Kotlin
    >>> val x: String? = null
    >>> var y: String = x
    ERROR: Type mismatch: inferred type is String? but String was expected
    
    >>> fun strLen(s: String): Int = s.length
    >>> strLen(x)
    ERROR: Type mismatch: inferred type is String? but String was expected
    ```
* What can you do with `nullable` variable is to compare it with null and use it as you want. Safely.
    ```Kotlin
    fun strLenSafe(s: String?): Int =
        if (s != null) s.length else 0
    ```
* There is no such thing as `nullable` and `non-null` types at runtime. All this information is used only during compilation, so there is no runtime overhead.

### Safe call operator: "?."
To prevent the code from getting more and more verbose with `null` checks and `Optional` wrappers, Kotlin provides a safe-call operator: `?.`. It combines a `null` check and a methodcall into a single operation.

```Kotlin 
s?.toUpperCase()
``` 
Is the same as:

```Kotlin 
if (s != null) s.toUpperCase() else null
```

![Safe call operator](/typesystem/safe_null_operator.png)

* The result type of such invocation is `nullable`
* Safe calls can be used for accessing properties
    ```Kotlin
    class Employee(val name: String, val manager: Employee?)
    
    fun managerName(employee: Employee): String? = employee.manager?.name
    ```
    ```
    >>> val ceo = Employee("Da Boss", null)
    >>> val developer = Employee("Bob Smith", ceo)
    >>> println(managerName(developer))
    Da Boss
    >>> println(managerName(ceo))
    null
    ```
* You can chain safe-calls
    ```Kotlin
    class Address(val streetAddress: String, val zipCode: Int, val city: String, val country: String)
    
    class Company(val name: String, val address: Address?)
    
    class Person(val name: String, val company: Company?)
    
    fun Person.countryName(): String {
        val country = this.company?.address?.country
        return if (country != null) country else "Unknown"
    }
    ```
 ### Elvis operator "?:"
 You can provide default values instead of `null`
 ```Kotlin
val a: String? = "123"
val b: String = a ?: ""
 ```
 * The Elvis operator works with `return` and `throw` as its right side
    ```Kotlin
    fun openSomeScreen(activity: Activity?) {
        val activity = activity :? return
        
        startActivity(Intent(activity, SomeClass.class))
    }
    ```
### Save casts: "as?"
`as` throws `ClassCastException` when you try to cast something with the wrong class.  You can check it with the `is` operator, but Kotlin has a sugary handy operator so you can write less code.

![as? operator](/typesystem/as_operator.png)

### Not-null assertions: "!!"
Even with the `null-safe` environment, Kotlin still allows you to shoot yourself in the leg, but it makes you very very aware of doing it.

```Kotlin
fun ignoreNulls(s: String?) {
    val sNotNull: String = s!!
    println(sNotNull.length)
}

>>> ignoreNulls(null)
Exception in thread "main" kotlin.KotlinNullPointerException
at <...>.ignoreNulls(07_NotnullAssertions.kt:2)
```
* Kotlin throws an exception not where you used the value. It generates an assertion before the usage and that's where it fails.

### The "let" function
* `?.let { }`
    ```Kotlin
    fun doStuffToString(s: String?) {
        s?.let {
            println(it.length) // you can be sure that it is not null
        }
    }
    ```
* `.let { }`

    ```Kotlin
    fun doStuffToString(s: String?) {
        s.let {
            val l: Int = s?.length ?: 0
        }
    }
    ```
### Late-initialized properties
Kotlin requires you to initialize all properties in the constructor or in the `init` block. However sometimes you can't do that. For example if you use a DI framework or you're writing tests. In that case you have to provide null values in for those properties and mark them as `nullable`. Actually you don't have to.
```Kotlin
class MyTest {
    private lateinit var myService: MyService
 
    @Before fun setUp() {
        myService = MyService()
    }
}
```
* It works only with `var`s

### Nullability of type parameters
* All type parameters of functions and classes in Kotlin are nullable.
* Any type, including a nullable type, can be substituted for a type parameter
    ```Kotlin
    fun <T> printHashCode(t: T) {
        println(t?.hashCode())
    }
    ```
### Nullability and Java
* Platform type
    Types of which Kotlin doesn't have nullability information are called Platform types.
    All types used in the Java code are considered as Platform types. If the Java code has the `@Nullable` and `@NonNull` annotations Kotlin will have nullability information but in all other cases it won't.
* If your project has both Java and Kotlin, beware that eventhough you defined something as `non-null`, still a `null` value can be assigned to it if it comes from Java. The good thing is that you'll get a meaningful runtime exception if that happens.

## Primitive and other basic types
### Primitive types: Int, Boolean, etc.
* Kotlin doesn't distinguish between primitive types and wrapper ones. You always use the same:
    * `Byte, Short, Int, Long`
    * `Float, Double`
    * `Char`
    * `Boolean`
* You always work the same way with the primitive types.
* You can use the methods of the Java wrapper class on the variable (For example you can use the Integer#compareTo(Integer) method on a normal `Int` variable)
* The compiler decides whether to use wrappers or not and it does use them only if necessary. In most cases it will use the Java primitive types.

### Nullable primitive types: Int?, Boolean?, etc.
Since `nullable` variables can hold `null` values (duuh.), nullable primitive types are represented as their wrappers.

### Nullable conversions
* Kotlin doesn't convert numbers from one type to another implicitly
    ```Kotlin
    val i = 1
    val l: Long = i // ERROR
    ```
    You have to do this explicitly
    ```Kotlin
    val i = 1
    val l: Long = i.toLong()
    ```
 * Conversion functions are defined for every primitive type except `Boolean`
    ```Kotlin
    val x = 1L
    val l = listOf(1, 2, 3)
    val b = x.toInt() in l
    
    x in l // ERROR
    ```
