# Android Framework

![](/android/components/res/comp-1.png)

## Intent

Intent - намерение выполнить какое-то действие.

Бывают:

- явные - Пример: явно указываем класс активити, которую хотим запустить.

- неявные - указываем некоторое действие, кот. хотим совершить. При этот система сама предложит варианты.
Пример: открытие ссылки

```Kotlin
val intent = Intent(Intent.ACTION_VIEW, url)
startActivity(intent)
```

Как система понимает, кому предложить выполнить действие? - Через интент фильтр, котороый задается в манифесте.

## Activity

Предоставляет некоторое окно для отрисовки интерфейса и взаимодействия с пользователем.

ЖЦ активити как набор состояний:

- Created - активити еще не видна
- Visible - видна пользователю, но взаимодействовать с ним еще не может
- Resumed - можно взаимодействовать

![](/android/components/res/comp-2.png)

onCreate - подготовка вьюшек
onStart()/onStop(), onResume()/onPause - захват/освобождение каких-то ресурсов

Пример 1: играем видео. В мультиоконном режиме хотим чтобы оно проигрывалось дальше. Тогда стоит использовать onStart()/onStop()

Пример 2: работа с камерой. В мультиоконном режиме хотим чтобы захват камеры остановился. Тогда стоит использовать onResume()/onPause

onDestroy() - может вызываться не всегда:

- активити в состоянии onStop(). Системе нужна память и она грохает активити. Тогда onDestroy() может не вызваться

### Изменение конфигурации

- поворот экрана
- смена локали
- подключение внешней клавиатуры
- ...

Что происходит с активити:

![](/android/components/res/comp-3.png)

Когда меняется конфигурация, нужно адаптироваться к ней, получая новые ресурсы.

### Сохранение состояния

- persistent storage (БД, файловое хранилище)
- ViewModel
- onSaveInstanceState

### onSaveInstanceState

складываем данные в Bundle - примитивы, Parcelable, Serializable

Serializable:

- объект сериализуется использую рефлексию

Parcelable:

- специально для андроид
- необходимо реализовать методы либо пометить спец аннотацией @Parcelize

Обычно состояние восстанавливают в onCreate()

onSaveInstanceState не поможет если:

- crash
- finish()
- onBackPressed()
- Вручную удаляем из списка запущенных приложений

Некоторые вью сами умеют сохранять свое состояние (нужно указать id вью!):

- EditText
- RecyclerView
- ...

Можно ли не пересоздавать активити при смене конфигурации?

- Можно. Но мы теряем адаптивность к сменам конфигурации.

![](/android/components/res/comp-4.png)

### Tasks and BackStack

На каждый процесс - свой BackStack

![](/android/components/res/comp-5.png)

LaunchMode - как активити будет себя вести в BackStack

- **standart**

![](/android/components/res/comp-6.png)

- **singleTop** - новый инстанс не создается, если активити на вершине стека. При этом, прилетит Intent и вызовется колбек onNewIntent()

![](/android/components/res/comp-7.png)

- **singleTask** - если активити уже в стеке - уничтожает все активити выше. передает Intent

![](/android/components/res/comp-8.png)

- **singleInstance** - активити будет единственной в системе. Такое поведение часто применяют при разработке лаунчера.

![](/android/components/res/comp-9.png)

Как задать LaunchMode?

- в манифесте
- при запуске активити через Intent.addFlags()

## Fragment

- Кусок UI на весь или часть экрана
- Удобное разделение логики
- Удобная навигация

Основные классы:

- Fragment
- FragmentManager
- FragmentTransaction

Добавление фрагмента:

```Kotlin
val fragment = SomeFragment()
supportFragmentManager
    .beginTransaction()
    .add(R.id.some_container, fragment, TAG) //replace, remove, hide, show
    .commit()
```

В рамках одной транзакции можно работать сразу с несколькими фрагментами.

ЖЦ фрагмента:

```Kotlin
onAttach()

onCreate()

onCreateView()

onStart()

onResume()

onPause()

onStop()

onDestroyView()

onDestroy()

onDetach()
```

onAttach и onDetach вызываются, когда мы создаем фрагмент и добавляем во FragmentManager

onCreateView и onDestroyView -  методы жц вью, привязанной к фрагменту

Передача данных во фрагмент:

- передача в конструктор - не следует так делать, тк экземпляр фрагмента также создается FragmentManager-ом, когда пересоздается активити. Он знает про конструктор по-умолчанию.
- setArguments(bundle: Bundle)

Если активити пересоздается, то не нужно выполнять транзакции еще раз, тк FragmentManager сделает это за нас.

retainInstance = true (Deprecated - use ViewModel) - фрагмент не будет уничтожен при пересоздании активити. Похожий механизм использует вьюмодель.

## Context

Context - God object
Context предоставляет:

- информацию о приложении
- доступ к ресурсам
- выполнение операций (запуск активити итп)
- доступ к системному АПИ (Context.getSystemService(Context.ALARM_SERVICE))

**Application Context** - часто не предоставляет информацию, связанную непосредственно с UI (тема итп)

**Activity Context**

## Service

Service - фоновое выполнение больших задач.
Объявление в манифесте.
Запуск:
Context.startService()

ЖЦ сервиса:

- onCreate()
- onStartCommand() - вызывается каждый раз при вызове Context.startService(). Происходит на главном потоке.
- onDestroy()

Остановка сервиса:

- вызов stopService()
- передача интента, несущего в себе команду к остановке и остановка сервиса изнутри командой stopSelf()
- сервис может сам завершить себя, если понимает, что возложенная на него задача выполнена

Виды сервисов:

- Foreground - есть уведомление. Настройку нужно сделать самостоятельно.
- Background - такой сервис будет быстро завершен.

IntentService - работает на фоновом потоке. В Android 11 помечен Deprecated. Рекомендуется использовать WorkManager.

## WorkManager

WorkManager предназначен для гарантированного выполнения короткой фоновой задачи. Доступен с АПИ 14. Это возможно потому что под капотом WorkManager использует на разных версиях разные технологии, в зависимости от их доступности.

Основные классы:

- Worker - описываем что нужно сделать
- WorkRequest - запрос на выполнение задачи
- WorkManager - управляет запросами

При создании WorkRequest, можно задать большое количество условий для выполнения запроса.

```Kotlin
val constraints = Constraints.Builder()
    .setRequiredNetwork(NetworkType.CONNECTED)
    .setRequiresCharging(true)
    .build()

val someWork = OneTimeWorkRequest.Builder(SomeWorker::class.java)
    .setConstraints(constraints)
    .build()
```

### Doze mode & Standby

WorkManager решает задачи по работе в режимах Doze mode & Standby.

**Doze mode:**

Когда телефон некоторое время не используется, система начинает ограничивать фоновые задачи в приложениях для уменьшения скорости разряда батареи.

![](/android/components/res/comp-13.png)

**Standby:**

Ограничение "простаивающих" приложений (которые долго не запускааем).

## BroadcastReceiver

- получает события извне приложения, например - системные, события из уведомлений, из виджетов
- получает и передает события между компонентами приложения

В последних версиях андроида наложилось следующее ограничение:

Ряд событий не будет обработан, если просто зарегистрировать BroadcastReceiver в манифесте. Чтобы их получать, нужно явно инициализировать BroadcastReceiver в коде активити.

Ограничения: время работы - несколько секунд

## Content Provider (see [[content-provider]])
