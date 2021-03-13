[содержание](../readme.md)

# UI-тестирование c Kakao

Основано на [этой][1] статье.

В этой лекции вы узнаете, как использовать **Kakao** для написания UI-тестов. UI-тесты на Android обычно пишутся с использованием *Espresso*. **Kakao** - это библиотека, предоставляющая доменный язык (DSL) для написания выразительных тестов Espresso.

>Тесты на Espresso выглядят не очень красиво. Когда вам нужно протестировать кейс, отличающийся от простого клика по кнопке, ваш тестовый код превращается в месиво, сложно поддающееся чтению.
>```kt
>@Test
>public void espressoTest() {
>  onView(allOf(allOf(withId(R.id.label_bf_hotelname), 
>        isDescendantOfA(withId(R.id.custom_view_trip_review))), 
>        isDescendantOfA(withId(R.id.contentView))))
>        .check(matches(withEffectiveVisibility(View.VISIBLE)));
>}
>```

## Espresso и Kakao

**Espresso** - это платформа тестирования с открытым исходным кодом от Google. Они создали его для предоставления API для создания тестов пользовательского интерфейса с учетом следующих характеристик:

* Маленький
* Предсказуемый
* Простой в освоении API

**Kakao** - это библиотека, построенная поверх Эспрессо. Agoda, которая создала более тысячи автоматизированных тестов для своей кодовой базы, разработала ее, когда они поняли, что читаемость кода их тестов была довольно низкой при использовании Эспрессо. Они разработали эту библиотеку преследуя следующие преимущества:

* Удобочитаемость
* Возможность повторного использования
* Расширяемый DSL

Перепишем на Какао тесты из прошлой лекции.

## Настройка

В зависимости добавьте:

```
androidTestImplementation 'com.agoda.kakao:kakao:2.0.0'
```

## Создание UI-тестов

### Основы

UI-тесты включают в себя следующие шаги:

* Поиск компонента (view): во-первых, вы будете искать view, в котором вы будете выполнять действия или проверять состояния. Для этого используется ViewMatchers. Как правило, вы ищете view по его идентификатору.

* Выполнение действия: найдя view, вы выполняете над ним какое-то действие, например клик или жест. Это ViewAction. Этот шаг необязателен, и он нужен вам только в том случае, если вы хотите выполнить какое-то действие.

* Проверка: наконец, вы проверите что-то на view. Например, вы можете проверить, виден ли он, изменилось ли положение или изменился ли текст или цвет. Это - ViewAssertions.

### Создание первого UI-теста

Первый тест: ввести в поле ввода текст и проверить что он там появился (testTypeLogin)

Для начала нужно настроить экран (screen):

Kakao требует класс, который наследуется от класса Screen. На этом экране вы добавляете view, участвующие в тестах. Это может какая-то часть или весь пользовательский интерфейс.

```kt
class MainScreen: Screen<MainScreen>() {
    val loginEdit: KEditText = KEditText { withId(R.id.loginEdit) }
}
```

Приведенный выше код настраивает screen (часть) для экрана авторизации. В нём у нас есть ссылка на поле ввода логина (loginEdit).

Мы объявляем иерархию элементов интерфейса, используя классы, которые предоставляет библиотека. Все эти классы (KView, KTextView, KButton) — пустые классы, которые наследуют логику интерфейсов: действий (Actions) и утверждений (Assertions).

Теперь мы можем написать тест:

```kt
// 1
@LargeTest
class MainActivityTest{
    // 2
    @Rule @JvmField
    val activityRule = ActivityTestRule(MainActivity::class.java)

    // 3 
    val screen = MainScreen()

    // 4
    @Test
    fun kakaoTestTypeLogin(){
        // 5
        screen {
            loginEdit.typeText("login")
            loginEdit.hasText("login")
        }
    }
}    
```

Что делает этот код:

1. Аннотация ``@LargeTest``. Она не обязательна, но рекомендуется её использовать.

2. Аннотация ``@Rule`` и получение экземпляра класса ActivityTestRule. Здесь стартует активность, которую мы будем тестировать.

3. Получаем экземпляр нашего экрана (screen)

4. Аннотация ``@Test`` позволяет *test runner* (запускальщику тестов) определять тестовые методы.

5. Используя объект screen, вводим текст в поле логина и проверяем что он там появился.

### Добавление других тестов

Во-первых, полностью опишем интерфейс:

```kt
class MainScreen: Screen<MainScreen>() {
    val loginInputLayout: KTextInputLayout = KTextInputLayout{ withId(R.id.loginInputLayout) }
    val passwordInputLayout: KTextInputLayout = KTextInputLayout{ withId(R.id.passwordInputLayout) }

    val loginEdit: KEditText = KEditText { withId(R.id.loginEdit) }
    val passwordEdit: KEditText = KEditText { withId(R.id.passwordEdit) }
    
    val logInButton: KButton = KButton { withId(R.id.logInButton) }
}
```

Добавились классы KTextInputLayout и KButton. Теперь мы можем проверить попытку логина с пустым полем *логин*:

```kt
@Test
@Throws(Exception::class)
fun kakaoEmptyLogin(){
    screen {
        with(loginEdit ) {
            isVisible()
            isFocusable()
            typeText("")
        }

        logInButton.click()

        loginInputLayout.hasError(AuthPresenter.ERROR_EMPTY_LOGIN)
    }
}
```

>Остальные тесты написать самомтоятельно на лабе

### Проверка запуска Intent

Проверим, что после успешной авторизации открывается следующая активность:

Перед выполнением теста нужно добавить правило для тестирования интентов:

```kt
@Rule @JvmField
var rule = IntentsTestRule(AfterLogin::class.java)
```

```kt
@Test
@Throws(Exception::class)
fun kakaoSuccessAuth() {
    // 1
    FuelManager.instance.client = object : Client {
        override fun executeRequest(request: Request): Response {
            val resp = """{"status":"OK"}""".toByteArray()
            return Response(
                request.url,
                body = DefaultBody.from( { ByteArrayInputStream(resp) }, {-1} )
            )
        }
    }

    // 2
    screen {
        loginEdit.typeText("login")
        passwordEdit.typeText("password")
        logInButton.click()
    }

    // 3
    val intent = KIntent {
        hasComponent(AfterLogin::class.java.name)
        //hasExtra("EXTRA_QUERY", query)
    }

    // 4
    intent.intended()
}
```

Что мы тут сделали:

1. Переопределили клиента для сетевых запросов для подделки ответа

2. Заполнили поля *логин*, *пароль* и нажали кнопку *логин*

3. Создали экземпляр KIntent, заодно проверяя существует ли нужный нам интент (AfterLogin). Дополнительно можно проверить параметры интента (hasExtra)

4. Проверяем, запущен ли этот интент

### Получение результатов выполнения Intent

В нашей задаче не используется, посмотрим на пример из оригинальной статьи:

```kt
class SomeScreen: Screen<SomeScreen>() {
    val ingredients = KEditText { withId(R.id.ingredients) }
}

@Test
fun choosingRecentIngredients_shouldSetCommaSeparatedIngredients() {
    screen {
      // 1
      val recentIngredientsIntent = KIntent {
        hasComponent(RecentIngredientsActivity::class.java.name)
        withResult {
          withCode(RESULT_OK)
          withData(
              Intent().putStringArrayListExtra(
                  RecentIngredientsActivity.EXTRA_INGREDIENTS_SELECTED,
                  ArrayList(listOf("eggs", "onion"))
              )
          )
        }
      }
      // 2
      recentIngredientsIntent.intending()

      // 3
      recentButton.click()

      // 4
      ingredients.hasText("eggs,onion")
    }
}
```

Что тут происходит:

1. Настраивается Intent, который возвращает результат

2. Инструкция Какао для stub (хз как перевести) настроенного интента

3. По клику кнопки запускается интент, результатов которого мы ждем

4. Проверяем результат. Обратите внимание, сюда мы попадем после того, как будет закрыт запущеный интент.

В оригинальной статье есть еще пара глав, но нам пока достаточно этого.

[содержание](../readme.md)

[_]: https://habr.com/ru/post/339664/ "Kakao — как сделать UI тестирование снова великим"
[1]: https://www.raywenderlich.com/1505688-ui-testing-with-kakao-tutorial-for-android-getting-started