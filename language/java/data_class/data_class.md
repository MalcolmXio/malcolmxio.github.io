# Data class

Предназначен для хранения данных.

Переопределен ряд методов:

- equals()
- hashCode()
- toString()

Добавлены методы:

- copy()
- componentN()

componentN() - используется для destructured declaration:

```Kotlin
data class Person(val name: String, val gender: String)
val person = Person("John", "male")
val (name, gender) = person
printLn("$name, $gender")
```

equals/hashCode - автоматически используют поля в конструкторе в реализации

Поля, объявленные не в конструкторе, не участвуют в реализации equals/hashCode

> :heavy_exclamation_mark:  
Один из аргументов data class-а может быть без переопределенного equals/hashCode (например: Throwable из Java). В таком случае, в дата классе придется переопределять equals/hashCode руками.
