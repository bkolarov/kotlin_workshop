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
    
    ```Kotlin
    fun strLen(s: String): Int = s.length
    ```
