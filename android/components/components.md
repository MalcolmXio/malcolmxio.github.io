# Android Framework

![](/android/components/res/comp-1.png)

## Intent

`Intent` - намерение выполнить какое-то действие.

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

`onCreate` - подготовка вьюшек
`onStart()`/`onStop()`, `onResume()`/`onPause` - захват/освобождение каких-то ресурсов

Пример 1: играем видео. В мультиоконном режиме хотим чтобы оно проигрывалось дальше. Тогда стоит использовать `onStart()`/`onStop()`

Пример 2: работа с камерой. В мультиоконном режиме хотим чтобы захват камеры остановился. Тогда стоит использовать `onResume()`/`onPause`

`onDestroy()` - может вызываться не всегда:

- активити в состоянии `onStop()`. Системе нужна память и она грохает активити. Тогда `onDestroy()` может не вызваться

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
- `ViewModel`
- `onSaveInstanceState`

### onSaveInstanceState

складываем данные в Bundle - примитивы, `Parcelable`, `Serializable`

`Serializable`:

- объект сериализуется использую рефлексию

`Parcelable`:

- специально для андроид
- необходимо реализовать методы либо пометить спец аннотацией `@Parcelize`

Существует расхожее мнение, что Serializable медленнее, чем Parcelable. Serializable использует рефлекшн и создает много дополнительных объектов, а в Parcelable разработчик сам указывает какие объекты сериализовать.
Исходя из этого умозаключения, рекомендуется всегда использовать Parcelable.

Но на самом деле такое сравнение Serializable и Parcelable не совсем честное. Дело в том, что в Serializable тоже есть режим «ручного управления».
Чтобы не использовать рефлекшн и задать сериализуемые поля вручную, нужно использовать методы writeObject() и readObject() в serializable-классе.
В этом случае Serializable работает быстрее, чем Parcelable.

[Исходный код](https://bitbucket.org/afrishman/androidserializationtest/src/default/) приложения, в котором измеряется время (де)сериализации Parcelable и Serializable на больших объектах.

Обычно состояние восстанавливают в `onCreate()`

`onSaveInstanceState` не поможет если:

- crash
- `finish()`
- `onBackPressed()`
- Вручную удаляем из списка запущенных приложений

Некоторые вью сами умеют сохранять свое состояние (нужно указать id вью!):

- `EditText`
- `RecyclerView`
- ...

Можно ли не пересоздавать активити при смене конфигурации?

- Можно. Но мы теряем адаптивность к сменам конфигурации.

![](/android/components/res/comp-4.png)

### Tasks and BackStack

На каждый процесс - свой BackStack

![](/android/components/res/comp-5.png)

`LaunchMode` - как активити будет себя вести в BackStack

- **standart**

![](/android/components/res/comp-6.png)

- **singleTop** - новый инстанс не создается, если активити на вершине стека. При этом, прилетит `Intent` и вызовется колбек onNewIntent()

![](/android/components/res/comp-7.png)

- **singleTask** - если активити уже в стеке - уничтожает все активити выше. передает `Intent`

![](/android/components/res/comp-8.png)

- **singleInstance** - активити будет единственной в системе. Такое поведение часто применяют при разработке лаунчера.

![](/android/components/res/comp-9.png)

Как задать `LaunchMode`?

- в манифесте
- при запуске активити через `Intent.addFlags()`

### Как запустить стек из нескольких активити?

Для старта стека из нескольких активити используется класс TaskStackBuilder.

```Kotlin
val taskStackBuilder = TaskStackBuilder.create(context)
    .addNextIntent(Intent(context, Activity1::class.java))
    .addNextIntent(Intent(context, Activity2::class.java))
    .addNextIntent(Intent(context, Activity3::class.java))

taskStackBuilder.startActivities()
```

После вызова метода startActivities(), стартует только activity3. Информация об activity1 и activity2 хранится в стеке. Когда пользователь нажимает «назад», или на activity3 вызывается метод finish(), создается и стартует activity2.

Этот механизм полезен для реализации роутинга при запуске приложения через deep link.

### task affinity

Если задать произвольный affinity, то это позволит приложению создать еще один activity стек. Таким образом, можно иметь несколько стеков для одного приложения.

Если пометить запуск активити флагом SINGLE_INSTANCE, то новый стек будет создан автоматически и туда будет помещена активити.

### Activity Result API

Если нам нужно стартануть из одной активити другую, а затем вернуться на предыдущую, при этом передав обратно какие-то данные, можно воспользоваться Activity Result API. `onActivityResult()` - deprecated

Шаги по использованию:

1. Создание контракта

```Kotlin
class MySecondActivityContract : ActivityResultContract<String, Int?>() {

   override fun createIntent(context: Context, input: String?): Intent {
       return Intent(context, SecondActivity::class.java)
           .putExtra("my_input_key", input)
   }

   override fun parseResult(resultCode: Int, intent: Intent?): Int? = when {
       resultCode != Activity.RESULT_OK -> null
       else -> intent?.getIntExtra("my_result_key", 42)
   }

   override fun getSynchronousResult(context: Context, input: String?): SynchronousResult<Int?>? {
       return if (input.isNullOrEmpty()) SynchronousResult(42) else null
   }
}
```

2. Регистрация контракта

```Kotlin
val activityLauncher = registerForActivityResult(MySecondActivityContract()) { result ->
   // используем result
}
```

3. Запуск контракта

```Kotlin
vButton.setOnClickListener {
   activityLauncher.launch("What is the answer?")
}
```

### Как ViewModel переживает пересоздание активити

Инстанс ViewModel попадает во ViewModelStore, если ViewModel создавалась через свойство-делегат `by viewModel`.

Класс ComponentActivity реализует метод getViewModelStore(): ViewModelStore интерфейса ViewModelStoreOwner.

ComponentActivity использует переопределенный метод Activity.onRetainNonConfigurationInstance() для сохранения объекта ViewModelStore. Этот метод вызывается между onStop() и onDestroy() и возвращает произвольный объект, который сохраняется системой во время пересоздания активити.

При вызове getViewModelStore(), ComponentActivity получает сохраненный ViewModelStore с помощью метода getLastNonConfigurationInstance().

## Fragment

- Кусок UI на весь или часть экрана
- Удобное разделение логики
- Удобная навигация

Основные классы:

- `Fragment`
- `FragmentManager`
- `FragmentTransaction`

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

`onAttach` и `onDetach` вызываются, когда мы создаем фрагмент и добавляем во `FragmentManager`

`onCreateView` и `onDestroyView` -  методы жц вью, привязанной к фрагменту

Передача данных во фрагмент:

- передача в конструктор - не следует так делать, тк экземпляр фрагмента также создается FragmentManager-ом, когда пересоздается активити. Он знает про конструктор по-умолчанию.
- `setArguments(bundle: Bundle)`

Если активити пересоздается, то не нужно выполнять транзакции еще раз, тк FragmentManager сделает это за нас.

`retainInstance = true` (Deprecated - use ViewModel) - фрагмент не будет уничтожен при пересоздании активити. Похожий механизм использует вьюмодель.

### Как сохраняется стек фрагментов?

В FragmentActivity перед уничтожением будет вызван метод onSaveInstanceState(), внутри которого сохраняется состояние всех фрагментов в стеке:

`mFragments` - ссылка на `FragmentManager` активити

```Java
@Override
protected void onSaveInstanceState(Bundle outState) {
     super.onSaveInstanceState(outState);
     Parcelable p = mFragments.saveAllState();
     if (p != null) {
         outState.putParcelable(FRAGMENTS_TAG, p);
     }
}
```

метод `FragmentManager.saveAllState()` вернет объект `FragmentManagerState` который содержит информацию обо всех активных фрагментах и стеке.

Далее, при пересохдании активити вызовется onCreate:

```Java
 @Override
 protected void onCreate(Bundle savedInstanceState) {
     mFragments.attachActivity(this, mContainer, null);
     // Old versions of the platform didn't do this!
     if (getLayoutInflater().getFactory() == null) {
         getLayoutInflater().setFactory(this);
     }
      
     super.onCreate(savedInstanceState);
     
     NonConfigurationInstances nc = (NonConfigurationInstances)
             getLastNonConfigurationInstance();
     if (nc != null) {
         mAllLoaderManagers = nc.loaders;
     }
     if (savedInstanceState != null) {
         Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
         mFragments.restoreAllState(p, nc != null ? nc.fragments : null);
     }
     mFragments.dispatchCreate();
 }
```

метод restoreAllState выглядит следующим образом:

```Java
void restoreAllState(Parcelable state, ArrayList<Fragment> nonConfig) {
    ...
    if (state == null) return;
    FragmentManagerState fms = (FragmentManagerState)state;
    ...
    mActive = new ArrayList<Fragment>(fms.mActive.length);
       if (mAvailIndices != null) {
          mAvailIndices.clear();
       }
       for (int i=0; i<fms.mActive.length; i++) {
           FragmentState fs = fms.mActive[i];
           if (fs != null) {
              Fragment f = fs.instantiate(mActivity, mParent);
              if (DEBUG) Log.v(TAG, "restoreAllState: active #" + i + ": " + f);
              mActive.add(f);
              // Now that the fragment is instantiated (or came from being
                // retained above), clear mInstance in case we end up re-restoring
                // from this FragmentState again.
                fs.mInstance = null;
            } else {
                mActive.add(null);
                if (mAvailIndices == null) {
                    mAvailIndices = new ArrayList<Integer>();
                }
                if (DEBUG) Log.v(TAG, "restoreAllState: avail #" + i);
                mAvailIndices.add(i);
            }
        }
    ...
}
```

Каждый объект FragmentState используется для создания фрагмента через рефлексию и дефолтный! конструктор. Вот почему важно не использовать другие конструкторы.

## Context

`Context` - God object

`Context` предоставляет:

- информацию о приложении
- доступ к ресурсам
- выполнение операций (запуск активити итп)
- доступ к системному АПИ (`Context.getSystemService(Context.ALARM_SERVICE)`)

**Application Context** - часто не предоставляет информацию, связанную непосредственно с UI (тема итп)

- не стоит стартовать активити с использованием Application Context, т.к. в таком случае будет создаваться новая задача - активити не попадет в текущий backStack.

- не стоит использовать Application Context для layout inflation, т.к. в таком случае будет использована дефолтная тема.

**Activity Context**

- создается при создании активити и уничтожается вместе с ней. Не стоит передавать в объект с ЖЦ, отличным от активити.

## Service

Service - фоновое выполнение больших задач. Объявляется в манифесте.

Виды сервисов:

- Foreground - есть уведомление. Настройку нужно сделать самостоятельно.
- Background - такой сервис будет быстро завершен.

IntentService - работает на фоновом потоке. В Android 11 помечен Deprecated. Рекомендуется использовать WorkManager.

### Started Service

Started Service запускается методом `Context.startService() / Context.startForegroundService()`. Intent должен быть явным.

ЖЦ Started Service:

- `onCreate()`
- `onStartCommand()` - вызывается каждый раз при вызове Context.startService(). Происходит на главном потоке. Возвращаемое значение из onStartCommand() должно быть одним из следующих:
  - START_NOT_STICKY - не пересоздает сервис, если его убила система
  - START_STICKY - пересоздает сервис, если его убила система, но последний Intent не будет доставлен
  - START_REDELIVER_INTENT - пересоздает сервис, если его убила система, последний Intent будет доставлен
- `onDestroy()`

Остановка сервиса:

- вызов `stopService()` из другого компонента
- передача интента, несущего в себе команду к остановке и остановка сервиса изнутри командой `stopSelf()`
- сервис может сам завершить себя, если понимает, что возложенная на него задача выполнена

### Bound Service

Bound Service привязывается к компоненту вызовом метода `bindService(Intent service, ServiceConnection serviceConnection, int flags)`.

Аргумент `serviceConnection` используется для взаимодействия с привязанным сервисом. После вызова `bindService()` у сервиса вызывается метод `onBind()`. Сервис должен передать объект `IBinder` в методе `onBind()`. Этот объект может использовать компонент через переопределенный метод `onServiceConnected()` в `serviceConnection`. Сервис стартует после вызова `bindService()`, если аргумент `flags` имеет значение `BIND_AUTO_CREATE`.

Для отвязывания компонента от сервиса используется метод `unbindService()`. У сервиса вызывается метод `onUnbind()`.
Если у сервиса больше нет привязанных компонентов, вызывается метод `onDestroy()`.

Компонент может быть привязан к сервису, запущенному методом startService(). В этом случае сервис относится сразу и к Started и к Bound. Особенность такой "двойной" службы в том, что даже при отвязке всех клиентов, служба продолжает свою работу и выполняется до тех пор, пока сама не остановит себя с помощью метода stopSelf(), или до тех пор, пока другой компонент не вызовет метод stopService().

### Взаимодействие с другим компонентом

Когда службе (привязанной или запущенной) необходимо отправлять сообщения привязанному клиенту, ей следует использовать что-то вроде LocalBroadcastManager (в том случае, если клиент и служба работают в одном процессе). Привязанные службы обычно не подключаются к привязанному клиентскому компоненту напрямую.

## WorkManager

Изначально это был инструмент только для отложенной фоновой работы, но в последних версиях Workmanager появилось несколько новых функций. Так, теперь появилась возможность запросить немедленное выполнение задачи, если приложение находится на переднем плане, и быть при этом уверенным в завершении этой работы. Например, приложение может продолжать обрабатывать фото, загружая их на сервер, даже если пользователь уже делает что-то другое.

Доступен с АПИ 14. Это возможно потому что под капотом WorkManager использует на разных версиях разные технологии, в зависимости от их доступности.

Типы работ:

- Immediate
- Long Running
- Deferrable

Основные классы:

- `Worker` - описываем что нужно сделать
- `WorkRequest` - запрос на выполнение задачи
- `WorkManager` - управляет запросами

При создании `WorkRequest`, можно задать большое количество условий для выполнения запроса.

```Kotlin
val constraints = Constraints.Builder()
    .setRequiredNetwork(NetworkType.CONNECTED)
    .setRequiresCharging(true)
    .build()

val someWork = OneTimeWorkRequest.Builder(SomeWorker::class.java)
    .setConstraints(constraints)
    .build()
```

WorkManager имеет свою базу данных, что позволяет сохранять информацию и статус работы в случае сбоя процесса или даже перезагрузки устройства. Это дает возможность восстановить статус выполнения работы и продолжить его с того места, на котором остановились, независимо от обстоятельств.

Как WorkManager гарантирует выполнение работы в фоне?

WorkManager связывает жизненный цикл процесса, в котором выполняется работа, с жизненным циклом foreground service. WorkManager по-прежнему осведомлен об этой работе, но foreground service полностью владеет жизненным циклом выполнения.

Так как foreground service требует показ уведомления пользователю, разработчики добавили API для WorkManager. Также WorkManager предоставляет API-интерфейсы, чтобы пользователь мог остановить выполнение работы прямо из уведомления.

[additional info](https://developer.android.com/guide/background/persistent)

### Doze mode & Standby

`WorkManager` решает задачи по работе в режимах Doze mode & Standby.

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

## [Content Provider](../content_provider/content_provider.md#content-provider)
