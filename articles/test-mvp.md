[содержание](../readme.md)

# Архитектура андроид приложений. Паттерн MVP. (Прошлая лекция)

Содрано [отсюда][1]

Вообще оригинальная лекция про архитектуру приложения и паттерн MVP, но в ее рамках пишется код, на который ссылаются следующие лекции по тестированию.

**Во-первых**, наша архитектура включает слой данных. Слой данных включает в себя объект *Repository*, который выполняет работу с серверными запросами и кэшированием и занимается первичной обработкой ошибок, а также средства для замены *Repository* при тестировании. **Во-вторых**, основой архитектуры является паттерн MVP. *Presenter* – это объект, который управляет отображением данных через специальный интерфейс View. *Presenter* также обращается к репозиторию за получением данных и занимается обработкой жизненного цикла. View – это интерфейс, содержащий методы для работы с UI, который реализуется в Activity или Fragment. Model – обычные модельки сущностей.

Давайте рассмотрим пример, как можно реализовать паттерн MVP для отдельного экрана, к примеру, экрана авторизации. Это простой экран, который состоит из двух полей ввода для логина и пароля и кнопки для инициации авторизации. Опишем этот экран более детально:

1. При открытии экрана нужно проверять текущее состояние авторизации, если пользователь уже авторизован, то открывать главный экран.
2. Если поле ввода для логина или для пароля пустое, то при попытке нажать кнопку должна отобразиться ошибка под пустым полем.
3. Если данные введены корректно, то при нажатии кнопки должен быть инициирован процесс авторизации (запрос на сервер).
4. Во время выполнения запроса нужно отображать прогресс бар, который необходимо скрывать после окончания запроса при любом исходе.
5. Если запрос выполнен успешно, то нужно открыть главный экран приложения.
6. Если во время выполнения произошла ошибка, нужно показать ошибку под полем ввода для логина.

<details>

<summary>
Разметка экрана (под спойлером). 

В разметке используется компонент TextInputLayout, который включает в себя TextEdit и позволяет отображать текст ошибки, причем со стилизацией:
</summary>

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/loginInputLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/loginEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Введите логин" />

        </com.google.android.material.textfield.TextInputLayout>

        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/passwordInputLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            tools:layout_editor_absoluteY="56dp">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/passwordEdit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:hint="Введите пароль"
                android:inputType="textPassword" />
        </com.google.android.material.textfield.TextInputLayout>

        <Button
            android:id="@+id/logInButton"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Логин"/>
    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```

</details>

>TextInputLayout не входит в набор базовых классов, поэтому при копировании разметки может возникнуть ошибка. Просто установите этот компонент (кнопка загрузить напротив компонента в палитре) и пересоберите проект (*Build* - *Reduild project* в меню Android Studio).

Создадим интерфейс (имеется в виду interface для класса) для *View*. Какие методы нужны *Presenter*-у для управления *View*? Отображение и скрытие процесса загрузки, отображение ошибок и открытие главного экрана (я еще добавил в интерфейс работу с токеном). Поэтому интерфейс *View* для экрана авторизации может выглядеть следующим образом:

```kt
/*
интерфейс LoadingView - отвечает за отображение и скрытие процесса загрузки. Весьма удобно вынести такие действия в базовый интерфейс, так как они нужны почти на каждом экране
*/
interface LoadingView {
    fun showLoading()
    fun hideLoading()
}

// интерфейс для экрана авторизации
interface AuthView : LoadingView {
    fun successLogin()
    fun showLoginError(error: String)
    fun showPasswordError(error: String)
    fun getToken(): String
    fun setToken(token: String)
}
```

Реализуем интерфейс *AuthView* в Activity для авторизации:

1. Добавляем интерфейс к описанию класса MainActivity

```kt
class MainActivity : AppCompatActivity(), AuthView {
                                          ^^^^^^^^
```

2. Реализуем методы интерфеса

```kt
//переход на следующую Activity (AfterLogin - пока просто пустая Activity, позже реализуем Repository)
override fun successLogin() {
    startActivity( Intent(this, AfterLogin::class.java) )
}

override fun showLoginError(error: String) {
    // сетевые запросы выполняются асинхронно, поэтому с UI работаем в основном потоке
    runOnUiThread {
        loginInputLayout.error = error
    }
}

override fun showPasswordError(error: String) {
    runOnUiThread {
        passwordInputLayout.error = error
    }
}

override fun showLoading() {
}

override fun hideLoading() {
}

// этого метода в оригинальной статье нет, тут мы просто считываем токен 
override fun getToken(): String {
    val myPreferences = getSharedPreferences("settings", MODE_PRIVATE)
    val token: String? = myPreferences.getString("token", "")
    return if(token==null) "" else token!!
}

override fun setToken(token: String) {
}
```

Нужно заметить, что методы очень простые, они не содержат какой-то особой логики и легко читаются, что удобно для разработчиков.

Теперь создадим класс **Presenter**, который будет управлять логикой экрана, процессом авторизации ~~и обработкой жизненного цикла (для обработки жизненного цикла будем использовать экземпляр LifecycleHandler из библиотеки RxLoader)~~. **Presenter**-у, разумеется, также передается экземпляр AuthView, которым он будет управлять ~~и специальный объект для управления жизненным циклом~~ (я пока LifecycleHandler не передаю, в текущей задаче он не нужен, дальше посмотрим что понадобится при тестировании):

Далее, при старте экрана авторизации **Presenter** будет проверять, авторизован ли пользователь или нет, и, если авторизован, то открывать главный экран:

```kt
/*
В оригинальной статье для чтения токена используется "левая" библиотека. 
Я лишние сущности добавлять не хочу, но т.к. напрямую использовать классы Андроид в этом классе нельзя (напоминаю, мы его пишем для тестов), то я добавил в интерфейс еще пару методов для работы с токеном, пусть приложение само разбирается где его хранить.
*/
class AuthPresenter(private val authView: AuthView){
    companion object {
        const val ERROR_EMPTY_LOGIN = "Не заполнено поле login"
        const val ERROR_EMPTY_PASSWORD = "Не заполнено поле password"
    }

    init {
        val token = authView.getToken()
        if(token.trim()!="")
            authView.successLogin()
    }
}
```

>Еще я добавил статические константы (companion object) для хранения текста ошибок. Правильнее их хранить в ресурсах приложения, но класс *AuthPresenter* не должен с ними напрямую работать. Правильным выходом было бы добавление в интерфейc *AuthView* еще и метода, который бы возвращал локализованный текст ошибки по ее коду, но это усложняет текст приложения и нам пока не принципиально. 

---

**Вбоквел**

В оригинальной лекции не описан класс **RepositoryProvider**: посмотрел в репозитории - там используются RxJava и Realm. Нам эти сущности не нужны, поэтому реализуем аналогичное поведение своими средствами (название я оставил то же, чтобы не путаться)

1. Нужен абстракный класс, содержащий методы для запросов к сети:

```kt
abstract class API(){
    abstract fun login(name: String, password: String): String
}
```

2. И потомок этого класса:

```kt
class DefaultAPI(): API() {
    override fun login(name: String, password: String): String = runBlocking {
        // запрос выполняем асинхронно (в корутине)
        val res = GlobalScope.async{
            // синхронный запрос к севреру
            val (_, _, result) = Fuel
                .post(
                    "http://192.168.1.153:8080/login",
                    listOf("login" to name, "password" to password)
                )
                .responseString()

            // результат последнего выражения будет результатом выполнения асинхронного блока
            when (result) {
                is Result.Failure -> throw Exception( result.error.message )
                is Result.Success -> {
                    // ответ сервера будет результатом асинхронного блока
                    result.get()
                }
            }
        }

        // метод await ЖДЕТ выполнения асинхронного блока (этот метод можно использовать только в suspend функциях)
        return@runBlocking res.await()
    }
}
```

Таким образом метод login либо вернет ответ сервера (асинхронно), либо выбросит исключение

3. И реализуем фабрику, которая возвращает экземпляр класса API (или его наследника)

```kt
class RepositoryProvider {
    companion object {
        var repository: API? = null
            get(){
                if(field==null) field = DefaultAPI()
                return field
            }
    }
}    
```

Рассмотрим подробнее этот класс:
* он реализует шаблон **фабрика**, т.е. возвращает экземпляр класса API (или его наследника)
* по-умолчанию фабрика вернет реализацию DefaultAPI()

---

Теперь переходим к основному сценарию авторизации. Добавим метод *tryLogIn* в класс **AuthPresenter**:

```kt
fun tryLogIn(login: String, password: String) {
    if (TextUtils.isEmpty(login)) {
        authView.showLoginError(ERROR_EMPTY_LOGIN)
    } else if (TextUtils.isEmpty(password)) {
        authView.showPasswordError(ERROR_EMPTY_PASSWORD)
    } else {
        // показ прогресс-бара
        authView.showLoading()

        try {
            // в оригинальной статье опущен момент с сохранением токена - это нужно делать тут
            val jsonResp = JSONObject(Factory.repository!!.login("qq", "ww"))
            if(jsonResp.has("status") && jsonResp.getString("status")=="OK") {
                authView.setToken("some token")                    authView.successLogin()
            } else {
                authView.showLoginError(ERROR_AUTH)
            }
        } catch (e: Exception){
            authView.showLoginError(ERROR_NETWORK)
        }
        authView.hideLoading()
    }
}
```

>Напоминалка: 
>* в зависимости добавляем `implementation 'com.github.kittinunf.fuel:fuel-android:2.2.1'`
>* в манифест `<uses-permission android:name="android.permission.INTERNET"/>` и  `android:usesCleartextTraffic="true"`
>* в импорт `import com.github.kittinunf.result.Result`

И в конструкторе MainActivity создаем экземпляр Presenter-а и вешаем обработчик на кнопку "Логин"

```kt
lateinit var mPresenter: AuthPresenter

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    mPresenter = AuthPresenter(this)

    logInButton.setOnClickListener { 
        mPresenter.tryLogIn(
            loginEdit.text.toString(), 
            passwordEdit.text.toString())
    }
}
```

Как мы и описывали раньше, **Presenter** проверяет логин и пароль на соответствие каким-то локальным условиям (в данном случае только на то, что они не пустые) и, либо показывает ошибку, либо выполняет запрос на сервер. Во время запроса он командует View отображать процесс загрузки, а после окончания – скрыть его. Он также обрабатывает результат и командует View либо показать ошибку, либо открыть следующий экран.

Таким образом, мы грамотно разделили код на части, отвечающие за логику и за UI, что упрощает написание кода, его понимание и дальнейшее тестирование.

[содержание](../readme.md)

[1]: https://www.fandroid.info/lecture-5-on-the-architecture-of-the-android-application-mvp-pattern/ "Лекция 5 по архитектуре андроид приложений. Паттерн MVP"