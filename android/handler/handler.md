# Handler/Looper/Message Queue

![](res/handler.png)

**Handler**:

- обработчик сообщений для потока
- предоставляет интерфейс для фоновых потоков для отправки сообщений в main thread.

**Looper** - диспетчер сообщений. Вытаскивает задачи из MQ и передает их на обработку Handler-у.

**Message Queue** - неограниченный по размеру буфер сообщений.

## Memory leaks

![](res/handler-2.png)

## Как не допустить утечек?

![](res/handler-3.png)

## Отправка сообщений на main thread

![](res/handler-4.png)

## Свой Worker Thread

![](res/handler-5.png)

или

![](res/handler-6.png)
