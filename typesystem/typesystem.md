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

