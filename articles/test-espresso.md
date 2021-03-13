[содержание](../readme.md)

# UI-тестирование

Продолжение [этой][1] статьи.

В начале прошлого занятия мы говорили о том, что интеграционное и системное тестирование в Android проводятся с точки зрения пользователя, то есть это в первую очередь тестирование UI. При этом грань между интеграционным и системным тестированием (тестирование одного экрана и тестирование всего приложения) для разработчика практически стерта, поэтому мы в общем будем употреблять слово UI-тестирование.

UI-тестирование подразумевает запуск приложения на реальном устройстве и проверку именно поведения UI, а не внутренних элементов. Мы не знаем и не обязаны знать, как реализован тот или иной экран. Мы открываем экран и выполняем на этом экране определенные действия так же, как это делал бы пользователь. Мы всегда знаем, к какому результату должны приводить те или иные действия пользователя, и можем их проверять опять же с точки зрения пользователя.

Относительно тестирования Android-приложений можно сказать очень важную вещь. Unit-тесты позволяют вам проверять то, что определенный метод вызывается при определенных действиях, а UI-тесты должны проверять то, что этот метод приводит к нужному для пользователя результату. Это звучит сложновато, поэтому проще привести в пример все тот же экран авторизации с одной из прошлых лекций. В Unit-тестах мы проверяем, что в случае нажатии кнопки и пустом поле для логина Presenter вызовет метод showLoginError, а в UI-тестах проверяется уже то, что метод showLoginError во View действительно отображает ошибку в TextInputLayout.

Мы начнем писать тесты с экрана авторизации, который мы уже протестировали с точки зрения логики и слоя данных, и заодно покажем, как можно использовать фреймворк Espresso.

В базовом представлении фреймворк Espresso позволяет находить View на экране, выполнять с ними какие-то действия и проверять выполнение условий. Простейший тест на Espresso может выглядеть следующим образом:

```kt
@RunWith(AndroidJUnit4::class)
class MainActivityTest{
    @Rule @JvmField
    val activityRule = ActivityTestRule(MainActivity::class.java)

    @Test
    fun testTypeLogin(){
        Espresso
            .onView(ViewMatchers.withId(R.id.loginEdit))
            .perform(typeText("MyLogin"))
    }
}
```

>Как обычно примеры сильно устарели, для современной AndroidStudio нужно в зависимости добавить:
>
>```
>androidTestImplementation 'androidx.test:runner:1.2.0'
>androidTestImplementation 'androidx.test:rules:1.2.0'
>```

Что происходит в этом тесте? Мы указываем в качестве *Runner* класс AndroidJUnit4, как уже делали для инструментальных тестов. Далее мы задаем правило (Rule), которое указывает, какую Activity запустить для тестов. И основное – в тестовом методе мы находим View с id loginEdit и над ней выполняем действие ввода текста "MyLogin".

Пока что в тестовом методе не выполняются никакие проверки, но мы можем их добавить. Проверим, что введенный текст действительно появился в поле ввода с id loginEdit:

```kt
@Test
fun testTypeLogin(){
    Espresso
        .onView(ViewMatchers.withId(R.id.loginEdit))
        .perform(typeText("MyLogin"))
        .check(matches(withText("MyLogin")))
}
```

То есть к предыдущему тесту добавляется проверка, что во View с id loginEdit отображается текст "MyLogin".

Это и есть общая схема всех тестов с Espresso:

1. Найти *View*, передав в метод onView объект Matcher.
2. Выполнить какие-то действия над этой View, передав в метод perform объект ViewAction.
3. Проверить состояние View, передав в метод check объект ViewAssertion. Обычно для создания объекта ViewAssertion используют метод matches, который принимает объект Matcher из пункта 1.

Espresso – это очень умный фреймворк, который грамотно проверяет все элементы, при этом он может ждать некоторое время, пока выполнится определенное условие, что очень удобно, так как не всегда элементы на экране появляются мгновенно.

Для каждого из этих объектов существует большое количество стандартных методов, которые позволяют покрыть подавляющее большинство сценариев проверки:

1. withId, withText, withHint, withTagKey, … – Matcher.
2. click, doubleClick, scrollTo, swipeLeft, typeText, … – ViewAction.
3. matches, doesNotExist, isLeftOf, noMultilineButtons – ViewAssertion.

И, разумеется, если вам не хватит стандартных объектов, вы всегда можете создать свои собственные, и мы рассмотрим, как это можно сделать.

А пока вернемся к тестированию экрана авторизации. Что мы могли бы проверить на этом экране с точки зрения UI:

1. Приложение запускается с пустыми полями ввода и кнопкой с корректным текстом.
2. При вводе пустого логина и нажатии на кнопку входа в поле ввода логина отображается ошибка.
3. При вводе пустого пароля и нажатии на кнопку входа в поле ввода пароля отображается ошибка.
4. При вводе корректных данных открывается следующий экран.
5. При вводе ошибочных данных в поле ввода логина отображается ошибка.

Первые 3 сценария не требуют от нас подмены ответов сервера и являются достаточно простыми в реализации. Поэтому начнем с них. Напишем тест, который проверит, что при старте отображаются поля ввода с пустыми текстами:

```kt
@Test
@Throws(Exception::class)
fun testEmptyInputFields(){
    Espresso
        .onView(withId(R.id.loginEdit))
        .check(matches(allOf(
            withEffectiveVisibility(Visibility.VISIBLE),
            isFocusable(),
            isClickable(),
            withText("")
        )))

    Espresso
        .onView(withId(R.id.passwordEdit))
        .check(matches(allOf(
            withEffectiveVisibility(Visibility.VISIBLE),
            isFocusable(),
            isClickable(),
            withInputType(InputType.TYPE_CLASS_TEXT or InputType.TYPE_TEXT_VARIATION_PASSWORD),
            withText("")
        )))
}
```

В данном методе мы для обоих полей ввода проверяем, что они отображаются, что на них можно навести фокус и нажать, и то, что в них отображается пустой текст. Для поля ввода пароля дополнительно проверяется, что они имеет тип ввода для пароля.

Аналогичным образом можно проверить работу кнопки "Логин":

```kt
@Test
@Throws(Exception::class)
fun testLogInButtonShown() {
    Espresso
        .onView(withId(R.id.logInButton))
        .check(matches(allOf(
            withEffectiveVisibility(Visibility.VISIBLE),
            isClickable(),
            withText(R.string.log_in)
        )))
}
```

Далее мы хотим проверить, что вводимый пользователем текст корректно отображается в полях ввода (например, мы не хотим, чтобы какой-то TextWatcher отслеживал изменения и менял текст логина / пароля в каких-то случаях):

```kt
@Test
@Throws(Exception::class)
fun testInputDisplayed() {
    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText("TestLogin"))

    closeSoftKeyboard()

    Espresso
        .onView(withId(R.id.passwordEdit))
        .perform(typeText("TestPassword"))

    closeSoftKeyboard()

    Espresso
        .onView(withId(R.id.loginEdit))
        .check(matches(withText("TestLogin")))

    Espresso
        .onView(withId(R.id.passwordEdit))
        .check(matches(withText("TestPassword")))
}
```

В этом тесте мы вводим данные для логина и пароля, а потом проверяем, что они отобразились в полях ввода. Здесь есть важный момент – после каждого ввода данных нужно закрывать клавиатуру после ввода данных. Иначе клавиатура может закрыть собой элемент, который вы хотите найти, и тогда Espresso выдаст ошибку.

Таким образом, мы написали различные тесты для случаев, которые проверяют корректность отображения и поведения View при старте экрана. Перейдем к следующим пунктам – проверка отображения ошибки при попытке входа, когда введен пустой логин или пароль. Напомним, что в такой ситуации мы отображаем ошибку с помощью TextInputLayout. В тесте мы вводим пустой текст в поле ввода логина и нажимаем на кнопку:

```kt
@Test
@Throws(Exception::class)
fun testLoginErrorDisplayed() {
    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText(""))

    closeSoftKeyboard()

    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())
}
```

И теперь возникает вопрос – как именно проверить тот факт, что в TextInputLayout отобразилась ошибка? Разумеется, можно получить экземпляр Activity, найти нужный TextInputLayout и проверить текущую ошибку:

```kt
@Test
@Throws(Exception::class)
fun testLoginErrorDisplayed() {
    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText(""))

    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())

    val inputLayout = activityRule.activity.findViewById(R.id.loginInputLayout) as TextInputLayout

    assertEquals(AuthPresenter.ERROR_EMPTY_LOGIN, inputLayout.error)
}
```

>В оригинальной статье текст ошибки берется из ресурсов: ``inputLayout.getResources().getString(R.string.error)``, но в таком случае придется и в интерфейсе **AuthView** делать методы для получения текста этих ошибок (напоминаю, что класс **AuthPresenter** не должен напрямую работать с классами Андроид). Я сделал эти сообщения статическими в классе **AuthPresenter** (это не очень хорошо с точки зрения мультиязычности, но нам пока сойдет):
>```kt
>companion object {
>   val ERROR_EMPTY_LOGIN = "Не заполнено поле login"
>}
>```

Но так никогда не нужно делать при тестирование с Espresso, так как это нарушает все принципы работы этого фреймворка. Вместо этого нам нужно описать свой Matcher для проверки. Тогда наш тест будет выглядеть вот так:

```kt
@Test
@Throws(Exception::class)
fun testLoginErrorDisplayed() {
    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText(""))

    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())

    Espresso
        .onView(withId(R.id.loginInputLayout))
        .check(matches(withInputError(AuthPresenter.ERROR_EMPTY_LOGIN)))
}
```

Метод withInputError – это статический метод для создания нашего объекта Matcher, который проверит, что ошибка, отображаемая в TextInputLayout, соответствует переданному тексту ошибки.

Как создать такой Matcher? Изначально, Matcher – это интерфейс, но его не нужно реализовывать напрямую, вместо этого следует либо наследоваться от класса BaseMatcher, либо использовать еще более продвинутый вариант с наследованием TypeSafeMatcher, как мы и поступим.

Создадим свой класс, который будет наследоваться от TypeSafeMatcher, и определим в нем статический метод withInputError, чтобы использовать его в дальнейшем:

```kt
class InputLayoutErrorMatcher(private val errorText: String): TypeSafeMatcher<View?>() {
    companion object {
        fun withInputError(errorText: String): InputLayoutErrorMatcher {
            return InputLayoutErrorMatcher(errorText)
        }
    }
}
```

И сейчас осталось реализовать два метода. Первый метод является главным и непосредственно определяет, выполняется ли проверяемое условие:

```kt
override fun matchesSafely(item: View?): Boolean {
    if (item !is TextInputLayout)
        return false

//  пример из оригинальной статьи - текст из ресурсов
//  val expectedError = item.getResources().getString(mErrorTextId)
//  return TextUtils.equals(expectedError, item.error)

    return TextUtils.equals(errorText, item.error)
}
```

В этом методе мы всего лишь получаем ожидаемую строку ошибки, строку ошибки, которая отображается в TextInputLayout, и сравниваем их.

И второй метод, который нам нужно реализовать при наследовании от TypeSafeMatcher – это вспомогательный метод для логирования describeTo. Когда вы получите ошибку при сравнении, Espresso выведет вам информацию о том, какие данные ожидались, а какие были получены. Такая информация поможет вам при отладке теста, поэтому методом describeTo не нужно пренебрегать. Реализуем его:

```kt
override fun describeTo(description: Description?) {
    description!!.appendText("with error: $errorText");
}
```

Таким образом, мы написали свой Matcher и убедились, что такими средствами можно очень удобно проверять состояние любых View и любые свойства. При этом мы полностью сохраняем парадигму работы Espresso.

Мы рассмотрели написание простых тестов на работу с UI-элементами, а теперь пора перейти к рассмотрению тестов на различные сценарии, связанные с серверными запросами, то есть сценарии удачной и неудачной авторизации.

//TODO: тут надо разобраться

>Конечно, как и во всех других видах тестов, для выполнения UI-тестов также стоит использовать моки для серверных запросов. Мы уже знаем все способы подмены серверных запросов в тестах, любой из них допустим для UI-тестов. Но мы будем пользоваться наиболее удобными средствами, а именно *product flavors* и подменой ответа на уровне OkHttp.
>
>Нужно добавить еще один небольшой момент. Подмена ответов при запросе выполняется почти мгновенно, но обычно запросы выполняются достаточно долго. Чтобы работа приложения на моках была максимально приближена к реальности (чего мы и хотим достичь), нужно добавить искусственную задержку в MockingInterctor:
>
>```java
>@Override
>public Response intercept(Chain chain) throws IOException {
>   Request request = chain.request();
>   String path = request.url().encodedPath();
>   if (mHandlers.shouldIntercept(path)) {
>       Response response = mHandlers.proceed(request, path);
>       int stubDelay = 500 + mRandom.nextInt(2500);
>       SystemClock.sleep(stubDelay);
>       return response;
>   }
>   return chain.proceed(request);
>}

По умолчанию у нас всегда проходит успешная авторизация, поэтому мы можем ввести любые непустые данные и нажать на кнопку входа:

```kt
@Test
@Throws(Exception::class)
fun testSuccessAuth() {
    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText("login"))
    closeSoftKeyboard()
    Espresso
        .onView(withId(R.id.passwordEdit))
        .perform(typeText("pass"))
    closeSoftKeyboard()
    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())
}
```

Единственный вопрос состоит в том, как проверить то, что была запущена какая-то Activity. В рамках Unit-тестов мы проверяли, что был вызван нужный метод. Но для данного случая такой подход неправильный, так как мы тестируем без знания реализации классов. Можно проверять, что открылся следующий экран по наличию каких-то данных на этом экран, но это тоже не очень правильно, так как мы тестируем только отдельный экран, а не все экраны вместе. К счастью, есть другой способ. В Android для запуска Activity используются Intent-ы. Espresso предоставляет возможность отслеживать запуск Activity через Intent, а также мокать такие вызовы. Для работы с Intent-ами в Espresso нужно подключить библиотеку в gradle:

```
androidTestImplementation 'androidx.test.espresso:espresso-intents:3.2.0'
```

И настроить методы с аннотациями Before и After:

```kt
@Before
fun setUp(){
    Intents.init()
}

@After
fun tearDown(){
    Intents.release()
}
```

Теперь мы можем проверить, что после нажатия кнопки входа происходит успешная авторизация и открывается следующий экран:

```kt
@Test
@Throws(Exception::class)
fun testSuccessAuth() {

    // подделываем ответ сервера
    FuelManager.instance.client = object : Client {
        override fun executeRequest(request: Request): Response {
            val resp = """{"status":"OK"}""".toByteArray()
            return Response(
                request.url,
                body = DefaultBody.from( { ByteArrayInputStream(resp) }, {-1} )
            )
        }
    }

    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText("login"))
    closeSoftKeyboard()
    Espresso
        .onView(withId(R.id.passwordEdit))
        .perform(typeText("pass"))
    closeSoftKeyboard()

    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())

    // добавили проверку 
    Intents.intended(hasComponent(AfterLogin::class.java.name))
}
```

Аналогичным образом проверяем, что в случае неудачной авторизации в TextInputLayout пользователю отображается ошибка:

```kt
@Test
@Throws(Exception::class)
fun testErrorAuth() {

    // симулируем ошибку авторизации
    FuelManager.instance.client = object : Client {
        override fun executeRequest(request: Request): Response {
            val resp = """{"status":"ERROR"}""".toByteArray()
            return Response(
                request.url,
                body = DefaultBody.from( { ByteArrayInputStream(resp) }, {-1} )
            )
        }
    }

    Espresso
        .onView(withId(R.id.loginEdit))
        .perform(typeText("login"))
    closeSoftKeyboard()
    Espresso
        .onView(withId(R.id.passwordEdit))
        .perform(typeText("pass"))
    closeSoftKeyboard()

    Espresso
        .onView(withId(R.id.logInButton))
        .perform(click())

    // проверяем наличие текста ошибки 
    Espresso
        .onView(withId(R.id.loginInputLayout))
        .check(matches(withInputError(AuthPresenter.ERROR_AUTH)))
}
```

Для симуляции ошибки авторизации в классе AuthPresenter добавлена обработка ответа:

```kt
is Result.Success -> {
    val jsonResp = JSONObject(result.get())
    if(jsonResp.has("status") && jsonResp.getString("status")=="OK") {
        // в оригинальной статье опущен момент с сохранением токена - это нужно делать тут
        authView.setToken("some token")
        authView.successLogin()
    } else {
        authView.showLoginError(ERROR_AUTH)
    }
}
```

Таким образом, мы протестировали различные сценарии экрана авторизации с точки зрения UI. Теперь мы можем не только убедиться, что логика этого экрана работает корректно, но также и то, что приложение запускается и работает так, как мы ожидаем.

TODO: реализовать репозиторий, как в оригинальной статье и дописать тестирование RecyclerView

[содержание](../readme.md)

[1]: https://www.fandroid.info/instrumental_testing_android_espresso_dagger2/ "Инструментальное и UI тестирование. Espresso. Dagger 2"
