# Система типов в Java и Kotlin

Тип - описание двух множеств:

- множество значений
- множество операций

## Типы в Kotlin

`Any` - супертип почти всех типов в Kotlin. Аналог `Object` в java.

В Any оставили 3 метода:

```Kotlin
toString()

hashCode()

equals()
```

`Unit` - аналог `void` из java. По умолчанию возвращается из функций, которые ничего не возвращают.

`Nothing` - когда нечего вернуть. Код дальше не исполняется.

```Kotlin
throw Exception(): Nothing

return: Nothing

while(true) {}: Nothing
```

Неявно `Nothing` наследуется от всех типов.

`Null`. `String` - подтип `String?`. Мы ничего не можем сделать с nullable типом, кроме как проверить на null и дальше работать как с обычным типом. Для этого в Kotlin есть удобный синтаксис.

`Flow-typing`

```Kotlin
val x: String?
if (x != null) {
    x: String //Компилятор знает, что x точно не null
}
```

## Boxing

Когда примитивный тип используется как референсный, он оборачивается в специальный класс (Boxing). Пример: записываем примитивный тип в переменную референсного типа.

```Kotlin
Object x;
x = 1; // Текущее значение?

//Integer в куче!
```

## Пулы примитивов

Для int: 0-299.

Для этих значений, если мы попытаемся завернуть примитив в Integer, то значение для этого типа будет всегда браться из пула.

```Kotlin
Integer.valueOf(100) === Integer.valueOf(100)

Integer.valueOf(300) !== Integer.valueOf(300)
```

Для строк

```Kotlin
"abc" === "abc"

"abc" !== String("abc".toCharArray())

"abc" === String("abc".toCharArray()).intern()
```
