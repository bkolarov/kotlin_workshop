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
