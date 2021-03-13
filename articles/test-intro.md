[содержание](../readme.md)

# Введение в архитектуру клиент-серверных андроид-приложений.

* [Введение в архитектуру android приложений](#Введение-в-архитектуру-android-приложений)
* [Основные задачи при разработке клиент-серверных приложений](#Основные-задачи-при-разработке-клиент-серверных-приложений)
* [Обработка смены конфигурации](#Обработка-смены-конфигурации)
    * [Запрет смены ориентации](#Запрет-смены-ориентации)
    * [Самостоятельная обработка](#Самостоятельная-обработка)
    * [Сохранение состояния в Bundle](#Сохранение-состояния-в-Bundle)
    * [Retain Fragments](#Retain-Fragments)
    * [Лоадеры](#Лоадеры)

## Введение в архитектуру android приложений

Прежде чем приступить непосредственно к изучению способов построения архитектуры клиент-серверных Android-приложений, было бы хорошо узнать, почему вообще эта тематика важна.

И этот вопрос логичен. **Во-первых**, пользователю совсем-совсем безразлична архитектура вашего приложения. Серьезно, кто из вас при использовании программ и приложений часто задумывается о том, сделали его по MVP или MVC? Ответ – никто. **Во-вторых**, работа с архитектурой требует дополнительных усилий: ее нужно создавать, в нее нужно вникать и учить людей работать по ней. Но чтобы создать более четкую картину для ответа на этот вопрос, нужно вернуться в относительно недалекое прошлое, а именно в 2007 и 2008 года, когда были выпущены соответственно первые версии устройств под iOS и Android.

Нужно признать, что Google успел отстать от Apple в плане выхода на мобильный рынок и это привело к некоторым последствиям, а именно к спешке при выходе первой версии Android. Нет ничего удивительного в том, что Google стремился в первую очередь доделать основные пользовательские функции, а забота об удобстве разработчиков была вторым приоритетом. Поэтому вместе с первой версией Android Google не предоставил разработчикам каких-то стандартных рекомендаций ни по разработке, ни по дизайну и UX. Что привело к тому, что каждый разработчик или каждая компания были вынуждены писать как хотели и как умели.

Конечно, нужно признать, что в дальнейшем Google проделал огромную работу по популяризации системы Android среди разработчиков. Но при этом изначальные проблемы так и не были до конца решены.

Какие же это проблемы? В плане дизайна понятно, что абсолютно разные стили приложений смущают пользователя системы Android, и ему бывает тяжело ориентироваться. Но что не так с отсутствием стандартов в самом коде? Ведь как было сказано чуть выше, пользователь никак не может знать, насколько хорош код вашего приложения, и это не влияет на его использование. Проблема заключается в том, что не все разработчики хорошо владеют паттернами проектирования и умеют разрабатывать хорошую архитектуру приложений. Если же не следовать четким принципам в архитектуре вы очень скоро получите код, который:

* *Невозможно поддерживать*. В коде будет много сложной логики, она не будет расположена в строго определенных классах, будет непонятно, как работает та или иная часть вашего приложения. Из этого следует, что при добавлении нового функционала вам придется либо долго и усиленно разбираться в написанном коде, рефакторить его и делать все правильно, либо сделать задачу кое-как, то есть, образно говоря, через костыли. В связи с тем, что не все разработчики понимают необходимость рефакторинга и умеют убеждать в этой необходимости руководство, и не каждый руководитель согласится отсрочить выход новой версии и понести дополнительные траты из-за рефакторинга, намного чаще выбирается второй вариант. Это часто приводит к ужасающим последствиям. Есть огромные приложения, состоящие из трех файлов Activity. Разумеется, каждая из этих Activity состоит из тысяч, а то и из десятков тысяч строк, что делает их абсолютно невозможными для чтения. Более того, каждый новый функционал, реализованный через костыли, является причиной дополнительных багов и крашей.

* *Невозможно протестировать*. Эта проблема плавно вытекает из первой. Вы не сможете писать модульные тесты, если все приложение – это один большой модуль. Более того, в силу особенностей написания тестов для Android-приложений на JVM, при большом количестве зависимостей от классов Android в тестируемых классах, вы не сможете писать тесты. А отсуствие тестов:
    * Дает вам гораздо меньше уверенности в том, что ваш код работает правильно.
    * Вы не сможете быстро проверить, что добавленные изменения не сломают работу остальных частей вашего приложения.

Такая ситуация продолжалась достаточно долго. Приложения под Android продолжались писаться в разных стилях с абсолютно разными подходами в дизайне и в архитектуре. Кто-то брал дизайн из системы iOS, а паттерны проектирования из Web-разработки (в частности, попытки использовать MVC в Android обязаны своему существованию именно Web-разработчикам, перешедшим в Android). И сложно сказать, почему не было никаких попыток исправить эту ситуацию, Android – это очень молодая система, и к моменту ее выхода все паттерны проектирования и архитектурные паттерны уже были широко известны.

В общем, все шло своим чередом до 2014 года, когда случилось сразу два важнейших события. Первое хорошо известно всем – это презентация концепции Material Design на Google I/O. Можно по-разному относиться к этой концепции, кто-то считает ее неудачной, кто-то говорит, что таким образом Google ограничивает свободу разработчиков в выборе дизайна. Но то, что появление этой концепции сильно улучшило ситуацию в среднем, – это бесспорно.

Понятно, что за конференцией Google I/O следят все и что Google приложил немало усилий в популяризации философии Material Design, так что Material Design был обречен на использование всеми. А вот другое знаковое событие произошло куда с меньшей популярностью, так как это была всего лишь статья. Это [статья][2] “Architecting Android… The clean way?” от Fernando Cejas. По сути эти всего лишь адаптация [принципов][3] Clean Architecture от “дядюшки Боба” (Роберта Мартина) для использования в Android. Эта статья дала огромный толчок (а вполне возможно, что это просто совпадение и статья вышла в тот момент, когда разработчики уже были готовы искать лучшие решения) в развитии архитектуры приложений.

Если говорить кратко (а подробнее мы посмотрим дальше по курсу), то хорошая архитектура должна позволять писать тесты для классов, содержащих бизнес-логику и должна строить модули приложения независимыми от почти всех внешних элементов. А если говорить еще проще, то ваш код должен быть тестируемым и его должно быть легко применять и приятно читать. Качество кода приложения можно даже замерить стандартной единицей измерения – количество WTF в минуту (из книги Роберта Мартина “Clean Code”).

Теперь мы можем примерно представить, что от нас требуется при построении архитектуры приложения и можем перейти непосредственно к рассмотрению всех тем!

## Основные задачи при разработке клиент-серверных приложений

Так в чем же заключается сложность создания клиент-серверных Android-приложений, которые бы удовлетворяли всем принципам, которые были описаны ранее? Есть 2 крупные проблемы, каждую из которых на самом деле можно разбить еще на большее число проблем:

* *Реализация клиент-серверного взаимодействия*. Казалось бы, в чем здесь проблема? Мы все умеем выполнять запросы к серверу с использованием различных средств, обрабатывать результат и показывать его пользователю. И да, и нет. Здесь существует масса факторов. **Во-первых**, нужно уметь корректно обрабатывать ошибки, которые могут быть самыми разными: от отсутствия интернета и неправильных параметров в запросе, до не отвечающего сервера и ошибках в ответе. **Во-вторых**, в вашем приложении может быть не один запрос, а много, и вполне возможна ситуация, что вам придется комбинировать результаты этих запросов сложным образом: выполнять их параллельно, использовать результат предыдущего запроса для выполнения следующего и так далее. **В-третьих**, и это самое неприятное – запросы могут занимать значительное время, а пользователь часто не самый терпеливый и тихий человек – он может крутить устройство (и тогда вы потеряете текущие данные в Activity), а может и вовсе закрыть приложение, и тогда вы можете получить рассинхронизацию в данных (когда на сервере данные обновились, а приложение не знает об этом и отображает некорректную или устаревшую информацию). И все это нужно каким-то образом решать.

* *Обеспечение возможности тестирования классов*, содержащих бизнес-логику приложения. Это также подразумевает под собой немало внутренних проблем. **Во-первых**, нужно обеспечить модульность классов. Это следует из самой сути и из самого названия Unit-тестов. Чтобы обеспечить модульность, нужно разделять классы по логическим слоям для каждого экрана. То есть вместо того, чтобы писать весь код, относящийся к одному экрану, в одной активити, нужно грамотно разделить его на несколько классов, каждый из которых будет иметь свою зону ответственности. **Во-вторых**, если говорить о тестах с помощью JUnit, то нужно понимать, что тестируемые таким образом классы должны содержать минимальное количество зависимостей от Android-классов, так как Java и ее виртуальная машина об этих классах не знает ничего (подробнее этот момент будет описан в лекции про тестирование). **В-третьих**, самая сложная логика приложения почти всегда связана с работой с данными от сервера. Мы должны протестировать различные возможные ситуации, такие как ожидаемый ответ сервера, ошибка сервера и разные ответы, приводящие к разному поведению приложения. Но при выполнении теста мы не можем по своему желанию “уронить” сервер или заставить его отдать нужные нам данные. К тому же, серверные запросы выполняются долго и асинхронно, а тесты должны работать последовательно. Все эти проблемы можно решить, если подменять реализацию сервера на определенном слое, к которому будут обращаться тестируемые классы. Все это также будет рассмотрено далее.

Эти проблемы и приходится решать при создании грамотной и правильной архитектуры, и это всегда не очень просто. Более того, иногда невозможно добиться желаемого результата, и у каждого способа есть как свои недостатки, так и достоинства. Рассмотрением всех этих способов мы и будем заниматься на протяжении курса, и после вы сможете решить, как именно вы хотите разрабатывать клиент-серверные приложения.

## Обработка смены конфигурации

Общеизвестно, что Activity пересоздается при каждом изменений конфигурации (например, при смене ориентации или языка). Пересоздание означает уничтожение Activity и запуск ее заново. Уничтожение в свою очередь подразумевает то, что все поля, которые вы хранили в Activity, будут уничтожены. Что это означает на практике? Это означает то, что, если вы при создании Activity получаете информацию с сервера, сохраняете ее в какое-то поле в Activity и отображаете пользователю, то при пересоздании Activity вы потеряете всю информацию, запрос начнет выполняться заново со всеми возможными последствиями:

* Пользователь не ожидает того, что ему снова покажется процесс загрузки, хотя он вроде бы ничего не делал, только повернул экран, к примеру.

* Вы можете не получить данные, например, в случае ошибки сервера. Это будет еще более странным поведением для пользователя, так как данные исчезли после поворота.

Так как же с этим бороться? И почему вообще возникла такая проблема при таком невинном действии?

Ответить на вопрос о том, почему такая проблема возникла, достаточно сложно. Очевидно, что было необходимо уничтожать все данные при изменении конфигурации, чтобы не показать пользователю некорректное состояние. Но наверняка был путь, который позволил бы уменьшить количество таких проблем, но, вероятно, он был слишком непростым для первых версий Android, а сейчас необходимо поддерживать обратную совместимость.

Раз сделать с такой ситуацией ничего нельзя, то приходится привыкать и к таким условиям. В самом деле, есть немало способов, как корректно сохранить и восстановить данные при пересоздании Activity. Рассмотрим их.

### Запрет смены ориентации

Разумеется, наиболее частой причиной пересоздания Activity по причине изменения конфигурации является смена ориентации. Поэтому достаточно многие разработчики, не желая иметь дела со всеми проблемами, связанными с обработкой жизненного цикла, жестко фиксируют ориентацию и дальше не думают об этой проблеме. Такая фиксация достигается за счет добавления флага в манифесте:

```xml
<activity
    android:screenOrientation="portrait"/>
```

Конечно, такой подход многое упрощает, но все же он не всегда приемлем. В принципе, существует немало приложений, которым достаточно только портретной ориентации, но это скорее исключение, чем правило. Часто пользователи работают в альбомной ориентации (особенно на планшетах) и заставлять их менять ее ради вашего приложения не очень хорошо. И ко всему прочему нужно понимать, что фиксированная ориентация не избавляет вас от проблем с пересозданием Activity, так как для этого есть и другие причины, а не только смена ориентации. Поэтому такое решение не может считаться идеальным.

### Самостоятельная обработка

Кроме того, можно не запретить изменение какой-то конфигурации (например, ориентации), а обрабатывать его самому. Для этого нужно указать флаг в манифесте с соответствующим значением:

```
<activity
    android:configChanges="orientation|keyboardHidden|screenSize"/>
``` 

В этом случае при изменении какой-то конфигурации система уведомит Activity о том, что произошло такое изменение и нужно его обработать. Для этого служит метод onConfigurationChanged в классе Activity:
 
```java 
@Override
public void onConfigurationChanged(Configuration newConfig) {
   super.onConfigurationChanged(newConfig);
   // handle new configuration
}
```

В каких-то случаях эта обработка не требуется. Но нужно понимать, что такая обработка также чревата последствиями, так как система не применяет альтернативные ресурсы автоматически (а это уже относится не только к языковым ресурсам, но и к привычным layout-land, к примеру). Поэтому это достаточно редкий вариант, но его нужно также иметь в виду.

### Сохранение состояния в Bundle

Android предоставляет нам способ сохранения состояния с последующим его восстановлением при пересоздании Activity. Здесь стоит обратить внимание на параметр savedInstanceState, который передается в методе onCreate в Activity. Этот экземпляр класса Bundle передается не просто так, он может хранить в себе различные поля, которые в него запишут. При первом запуске Activity этот параметр всегда будет равен null. При пересоздании Activity он уже не будет равен null, поэтому можно отследить, происходит ли первый запуск Activity или же это вызов после пересоздания, что весьма удобно. И теперь главное – вы можете сохранить в Bundle свои поля в методе onSaveInstanceState в классе Activity примерно следующим образом:

```java
@Override
protected void onSaveInstanceState(Bundle outState) {
   super.onSaveInstanceState(outState);
   outState.putSerializable(WEATHER_KEY, mCity);
}
```

И этот объект класса Bundle, в который вы сохранили какие-то значения, после пересоздания попадет в качестве параметра в метод onCreate, и уже оттуда вы сможете извлечь все данные. При таком подходе код для обработки смены состояния экрана выглядит следующим образом:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_weather);
   if (savedInstanceState == null) {
       loadWeather();
   } else {
       mCity = (City) savedInstanceState.getSerializable(WEATHER_KEY);
       showWeather();
   }
}
```

То есть мы проверяем, если Activity запускается в первый раз (словосочетание “в первый раз” здесь не очень подходит, поскольку Activity может запускаться несколько раз, но здесь понимается запуск не после пересоздания), то мы начинаем загружать информацию о погоде. Если же Activity пересоздается, то мы сохраняем информацию о погоде в методе onSaveInstanceState, а восстанавливаем в методе onCreate.

Тут нужно заметить важный факт – не всегда погода будет загружена до того, как Activity будет пересоздана. Поэтому код выше надо слегка модифицировать:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_weather);
 
   if (savedInstanceState == null || !savedInstanceState.containsKey(WEATHER_KEY)) {
       loadWeather();
   } else {
       mCity = (City) savedInstanceState.getSerializable(WEATHER_KEY);
       showWeather();
   }
}
 
@Override
protected void onSaveInstanceState(Bundle outState) {
   super.onSaveInstanceState(outState);
 
   if (mCity != null) {
       outState.putSerializable(WEATHER_KEY, mCity);
   }
}
```

Возможно, что этот способ не настолько удобен, но он хорошо работает, когда вы должны сохранить небольшие данные на каком-то одном экране. Но при этом нужно учитывать, что таким образом нельзя сохранять большие объекты или огромные объемы данных, так как их сериализация и восстановление занимает много времени, и из-за этого приложение будет работать медленно.

### Retain Fragments

Еще одним очень популярным и очень эффективным способом обработки смены конфигурации являются Retain Fragments. По сути это обычные фрагменты, для которых был вызван метод setRetainInstance:

```java
public class WeatherFragment extends Fragment {

   @Override
   public void onCreate(@Nullable Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setRetainInstance(true);
   }
}
```

Вызов этого метода меняет жизненный цикл фрагмента, а именно, он убирает из него вызовы onCreate и onDestroy при пересоздании Activity. Теперь при пересоздании Activity этот фрагмент не будет уничтожен, и все его поля сохранят свои значения. Но при этом остальные методы из жизненного цикла Fragment будут вызваны, так что не возникнет проблем с заменой ресурсов в зависимости от конфигурации. Поэтому нам только нужно добавить этот фрагмент при первом старте Activity и выполнять все запросы в нем, так как ему безразличны пересоздания Activity:

```java
@Override
 
protected void onCreate(@Nullable Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_weather_retain);
 
   if (savedInstanceState == null) {
       WeatherFragment fragment = new WeatherFragment();
       getSupportFragmentManager().beginTransaction()
               .replace(R.id.container, fragment)
               .commit();
   }
}
```

Во фрагменте в методе onViewCreated мы проверяем, если данные уже загрузились, то отображаем их, иначе начинаем загрузку данных и показываем процесс загрузки:

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setRetainInstance(true);
}
 
@Override
public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
   super.onViewCreated(view, savedInstanceState);
 
   if (mCity == null) {
       loadWeather();
   } else {
       showWeather();
   }
}
```

Кажется, что такой подход идеален. И в принципе, да, у него есть серьезные достоинства, по сравнению с предыдущими подходами:

* Можно сохранять даже большие и сложные объекты
* Не нужно сохранять данные вручную

Но при этом нужно понимать и ограничения такого подхода:

* Во-первых, нужно быть крайне аккуратными с сохранением в таком фрагменте любых ссылок на Activity / Context. При пересоздании Activity она уничтожается, но, если ваш фрагмент будет держать ссылку на эту Activity, то сборщик мусора не сможет утилизировать ее, и вы можете получить утечки памяти. А это в свою очередь накладывает определенные ограничения – вы уже не можете сохранять ссылку на Activity в методе onCreate такого фрагмента.

* Такой подход хорошо помогает против проблемы поворотов, но, к сожалению, он точно также привязан к текущему видимому экрану пользователя. И это значит, что, если пользователь решит закрыть приложение во время выполнения какого-то запроса, фрагмент будет уничтожен, и мы также потеряем нужные данные.

Поэтому и работа с retain фрагментами требует аккуратности и имеет свои минусы.

### Лоадеры

И последним компонентом, который мы рассмотрим, будут лоадеры. Несмотря на принципиальные различия в сути, с точки зрения проблемы обработки смены конфигурации, этот компонент очень похож на предыдущий: он точно также без потери данных переживает пересоздание Activity, он точно также управляется специальным классом (LoaderManager), как и фрагменты (FragmentManager).

Лоадеры будут использоваться и дальше в ходе курса, поэтому сейчас мы остановимся на них чуть подробнее.

Даже если смотреть по названию, лоадеры должны быть предназначены для загрузки чего-либо. А обычно мы загружаем данные – из базы или с сервера. Поэтому решение использовать лоадеры для задачи обеспечения клиент-серверного взаимодействия выглядит логично. Так что же такое лоадеры и как их использовать?

Лоадер – это компонент Android, который через класс LoaderManager связан с жизненным циклом Activity и Fragment. Это позволяет использовать их без опасения, что данные будут утрачены при закрытии приложения или результат вернется не в тот коллбэк. Разберем простейший пример (который хоть и простейший, но требует немало кода, это один из недостатков лоадеров). Создаем класс лоадера (для простоты он не будет грузить данные с сервера, а лишь имитировать загрузку):

```java
public class StubLoader extends AsyncTaskLoader<Integer> {
 
   public StubLoader(Context context) {
       super(context);
   }
 
   @Override
   protected void onStartLoading() {
       super.onStartLoading();
       forceLoad();
   }
 
   @Override
   public Integer loadInBackground() {
       // emulate long-running operation
       SystemClock.sleep(2000);
       return 5;
   }
}
```

Класс лоадера очень похож на класс AsyncTask-а  (впрочем, не зря же мы наследуемся от AsyncTaskLoader). Понятно, что в методе loadInBackground мы должны загрузить данные, а вот для чего нужен метод onStartLoading (и другие методы) мы разберем позже. А пока перейдем к использованию. В отличие от AsyncTask-а лоадер не нужно запускать вручную, это делается неявным образом через класс LoaderManager. У этого класса есть два метода с одинаковой сигнатурой:

```java
public abstract <D> Loader<D> initLoader(int id, Bundle args,  LoaderManager.LoaderCallbacks<D> callback);
 
public abstract <D> Loader<D> restartLoader(int id, Bundle args, LoaderManager.LoaderCallbacks<D> callback);
```

Про различие этих методов будет рассказано сразу после того, как мы рассмотрим их применение. Какие есть параметры у этих методов? Во-первых, вы можете запускать несколько лоадеров в одной Activity / Fragment, и вам нужно будет различать их. Для этого служит параметр id. Вторым параметром идет Bundle, с помощью которого вы можете передать аргументы для создания лоадера. И последний параметр является основным – это Callback для создания лоадера и получения результата его работы:
 
```java 
public interface LoaderCallbacks<D> {
 
   public Loader<D> onCreateLoader(int id, Bundle args);
 
   public void onLoadFinished(Loader<D> loader, D data);
 
   public void onLoaderReset(Loader<D> loader);
}
```

В методе onCreateLoader вы должны вернуть нужный лоадер в зависимости от переданного id и используя аргументы в Bundle. В методе onLoadFinished в параметре D вам придет результат работы лоадера. В методе onLoaderReset вы должны очистить все данные, которые связаны с этим лоадером.

Тогда давайте создадим экземпляр LoaderCallbacks для нашего лоадера:

```java
private class StubLoaderCallbacks implements LoaderManager.LoaderCallbacks<Integer> {
 
   @Override
   public Loader<Integer> onCreateLoader(int id, Bundle args) {
       if (id == R.id.stub_loader_id) {
           return new StubLoader(WeatherActivity.this);
       }
       return null;
   }
 
   @Override
   public void onLoadFinished(Loader<Integer> loader, Integer data) {
       if (loader.getId() == R.id.stub_loader_id) {
           Toast.makeText(WeatherActivity.this, R.string.load_finished, Toast.LENGTH_SHORT).show();
       }
   }
 
   @Override
   public void onLoaderReset(Loader<Integer> loader) {
       // Do nothing
   }
}
```
 
И теперь запустим лоадер, наконец-то подобравшись к сути:

```java 
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_weather);
   getSupportLoaderManager().initLoader(R.id.stub_loader_id, Bundle.EMPTY, new StubLoaderCallbacks());
}
```

Через 2 секунды после запуска Activity покажется тоаст. В принципе, ничего другого мы и не ждали. Но теперь главное – повернем устройство. Теперь по логике работа лоадера должна начаться заново и через 2 секунды покажется тоаст. Однако при этом мы видим, что тоаст показался мгновенно!

Вся магия заключается в методе initLoader и в классе LoaderManager. И теперь настала пора объяснить, как эта магия работает. Если лоадер еще не был создан (он не хранится в LoaderManager), то метод initLoader создает его и начинает его работу. Однако, если лоадер уже был создан (при первом запуске активити), то при повторном вызове initLoader, LoaderManager не пересоздаст лоадер и не будет его перезапускать. Вместо этого он заменит экземпляр LoaderCallbacks на новый переданный (что замечательно, ведь старый экземпляр был уничтожен вместе со старой Activity) и, если данные уже загрузились, передаст их в onLoadFInished. И при всем этом нам не нужно заниматься ручной проверкой на null для Bundle в onCreate. Если у нас на экране есть один запрос, то мы вполне можем при старте вызывать initLoader и не беспокоиться о пересоздании, что дает хороший уровень абстракции при обработке смены конфигурации.

Логичным будет вопрос о том, а что если мы все же хотим перезапустить выполнение лоадера (например, мы хотим обновить данные, а не получить предыдущий результат)? Для этого есть метод restartLoader, который всегда перезапускает выполнение лоадера. В остальном его работа абсолютно аналогична вызову initLoader.

Давайте теперь посмотрим, как это можно применить для решения нашей реальной задачи и реального запроса. Создадим лоадер, который будет загружать информацию о погоде:

```java
public class WeatherLoader extends AsyncTaskLoader<City> {
 
   public WeatherLoader(Context context) {
       super(context);
   }
 
   @Override
   protected void onStartLoading() {
       super.onStartLoading();
       forceLoad();
   }
 
   @Override
   public City loadInBackground() {
       String city = getContext().getString(R.string.default_city);
       try {
           return ApiFactory.getWeatherService().getWeather(city).execute().body();
       } catch (IOException e) {
           return null;
       }
   }
}
```

И вызовем его в Activity следующим образом:

```java
private void loadWeather(boolean restart) {
   mWeatherLayout.setVisibility(View.INVISIBLE);
   mErrorLayout.setVisibility(View.GONE);
   mLoadingView.showLoadingIndicator();
   LoaderManager.LoaderCallbacks<City> callbacks = new WeatherCallbacks();

    if (restart) {
        getSupportLoaderManager().restartLoader(R.id.weather_loader_id, Bundle.EMPTY, callbacks);
    } else {
        getSupportLoaderManager().initLoader(R.id.weather_loader_id, Bundle.EMPTY, callbacks);
    }
}
```

Обратите внимание на передаваемый флаг, который определяет, какой метод вызывать, initLoader или restartLoader. Даже на таком простом экране нам требуется вызов restartLoader, например, для обработки ошибки, когда пользователь хочет выполнить запрос повторно. Аналогичная ситуация возникает и при обновлении данных (например, Callback от SwipeRefreshLayout).

Нужно опять заметить важный плюс от использования лоадеров – мы нигде специально не пишем код для обработки пересоздания, что очень удобно.

Как уже говорилось выше, также очень важно понимать внутреннее устройство класса Loader. Если раньше особо не было другого пути, как изучать лоадеры, то сейчас ситуация изменилась. Сегодня технологии разработки под Android выросли очень серьезно, появилось множество библиотек для обеспечения асинхронности и работы с сетью (именно поэтому типичный путь разработчика сейчас выглядит следующим образом: Thread, AsyncTask, RxJava, минуя лоадеры). Но знать такой мощный компонент и уметь его использовать – это необходимо. Поэтому мы разберем и то, как лоадер устроен внутри.

Пока что мы наследовались всегда от класса AsyncTaskLoader, который обеспечивал работу в фоне. Но все же изначальным классом является именно класс Loader. При этом примечательно, что класс Loader не предоставляет никаких средств для обеспечения работы в фоне. И это не просто так. Лоадер в первую очередь предназначен для того, чтобы быть связанным с жизненным циклом Activity / Fragment. Для обеспечения же работы в фоне нужно либо использовать класс AsyncTaskLoader, либо использовать другие средства обеспечения многопоточности (например, Call из Retrofit, RxJava). И за такое решение нужно сказать большое спасибо разработчикам из Google. Ведь они позволили нам использовать свои средства для обеспечения многопоточности (иначе у нас, к примеру, не было бы возможности использовать RxJava в связке с лоадерами, чем мы займемся далее), при этом сохранив мощь лоадеров.

В классе Loader определено 3 основных метода, которые нужно переопределить для корректного написания своего лоадера. Это следующие методы:

```java
protected void onStartLoading() {
}
 
protected void onForceLoad() {
}
 
protected void onStopLoading() {
}
```

Метод onStartLoading вызывается в случае, когда нужно загрузить данные и вернуть их в Callback. На самом деле этот метод вызывается как результат вызова метода startLoading. Обычно метод startLoading вызывается классом LoaderManager, и нам нет нужды использовать его самостоятельно.

Аналогично метод onStopLoading служит для уведомления о том, что нужно остановить загрузку данных (запрос к серверу, к примеру), но при этом не нужно очищать данные.

Методы onStartLoading и onStopLoading вызываются соответственно при вызове методов onStart и onStop в Activity, но при этом они не вызываются в ситуации, когда Activity пересоздается, а только при сворачивании / разворачивании. И это очень хорошо, ведь мы должны останавливать загрузку данных, когда пользователь не находится на экране, чтобы не тратить заряд батареи.

Так что же, это означает, что при каждом сворачивании / разворачивании экрана процесс загрузки будет начинаться заново? Вовсе нет, но для этого нам придется немного поработать самим. Поля в лоадере не уничтожаются, а значит, мы можем проверить, завершился ли уже запрос, и, если да, то мы можем вернуть полученные данные.

Поэтому написание собственного лоадера обычно выглядит следующим образом. Во-первых, определяются поля лоадера, а это обычно результат загрузки, который мы хотим получить, и объект для выполнения запроса к серверу:

```java
public class RetrofitWeatherLoader extends Loader<City> {
 
   private final Call<City> mCall;
 
   @Nullable
   private City mCity;
 
   public RetrofitWeatherLoader(Context context) {
       super(context);
       String city = context.getString(R.string.default_city);
       mCall = ApiFactory.getWeatherService().getWeather(city);
   }
}
```

После этого переопределяется метод onStartLoading, в котором мы проверяем, завершился ли запрос (доступны ли сохраненные данные). Если данные есть, то мы сразу возвращаем их, иначе начинаем загружать все заново:

```java
@Override
protected void onStartLoading() {
   super.onStartLoading();
 
   if (mCity != null) {
       deliverResult(mCity);
   } else {
       forceLoad();
   }
}
```

Метод deliverResult возвращает данные в LoaderCallbacks. А метод forceLoad инициирует вызов метода onForceLoad, который мы упомянули, но обсудить не успели. По сути, этот метод служит только для удобства и логического разделения между методами жизненного цикла и методами для загрузки данных. В методе onForceLoad вы должны загрузить данные асинхронно и вернуть результат с помощью метода deliverResult.

```java
@Override
 
protected void onForceLoad() {
   super.onForceLoad();
   mCall.enqueue(new Callback<City>() {
 
       @Override
       public void onResponse(Call<City> call, Response<City> response) {
           mCity = response.body();
           deliverResult(mCity);
       }
 
       @Override
       public void onFailure(Call<City> call, Throwable t) {
           deliverResult(null);
       }
   });
}
```

И остался последний метод – onStopLoading, в котором мы должны остановить загрузку данных. Благо, с Retrofit это очень просто:

```java
@Override
protected void onStopLoading() {
   mCall.cancel();
   super.onStopLoading();
}
```

И это все – мы полностью изменили способ загрузки данных, но при этом не нужно ничего менять в UI-классах – потрясающий уровень абстракции!

Здесь можно закончить рассмотрение лоадеров, этого достаточно для дальнейшего изучения. Больше примеров и варианты работы с базой данных и обработкой ошибок можно найти в [статье][4].

Несмотря на все удобства, которые мы рассмотрели, и у класса Loader есть свои слабые места, и они почти всегда аналогичны проблемам с Retain Fragment. Например, при полном закрытии приложения в момент загрузки данных, вы также можете получить рассинхронизированное состояние. При этом отличие заключается в том, что лоадер специально предназначен для загрузки данных и предоставляет больший уровень абстракции, но при этом он требует больше кода.


[содержание](../readme.md)


[1]: https://www.fandroid.info/lektsiya-1-vvedenie-v-arhitekturu-klient-servernyh-android-prilozhenij-chast-1/ "Введение в архитектуру клиент-серверных андроид-приложений."

[2]: https://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/ "Architecting Android...The clean way?"

[3]: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html "The Clean Architecture"

[4]: https://habr.com/ru/company/e-legion/blog/265405/ "Android архитектура клиент-серверного приложения"
