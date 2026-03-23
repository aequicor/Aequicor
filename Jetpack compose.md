## Strong skipping mode

Compose имеет механизм пропуска функций ветвей при рекомпозиции путём проверки, что никакие параметры функции не изменились. Однако для этого требовалось, чтобы все параметры были иммутальные, т.е. они не должно изменять свои внутренние свойства.  Под такие требования подходят примитивные типы java или дата-классы с val параметрами и @Stable классы. 

Стабильные типы: 
1) примитивы (Int, String, Boolean, Float, Double)
2) Класс, где все свойства имеют тип val

Нестабильные типы: 
1) класс содержит var переменную
2) класс содержит нестабильный тип
3) Стандартные коллекции: List, Set, Map

Чтобы сделать класс содержащий нестабильный тип стабильными можно использовать аннотации: Immutable или Stable. С их помощью мы обещаем компиляторы, что данные классы не изменяются. Но есть небольшая разница

@Immutable, обещает компилятору, что никакое свойства класса не изменится никогда @Stable, обещает компилятору, что если свойство изменится, класс сообщит от этом. И последнее делается когда параметрами являются стейты. Также @Stable говорит компилятору, что `equals()` для этого лкасса всегда будет возвращать одинаковый результат для одних и тех же инстансов.



Пример использования @Immutable: 
```kotlin
@Immutable
data class MessageUiModel(
    val id: String,
    val text: String,
    val senderName: String,
    val attachments: List<String> // Без аннотации List сделал бы класс нестабильным
)
```

Пример использования @Stable:
```kotlin
@Stable
class ChatState {
    var isTyping by mutableStateOf(false)
    // Compose увидит изменение State и запустит рекомпозицию нужного куска
}
```


Это зачастую накладывает дополнительные расходы на создание стабильных оберток. Например, в компоуз нельзя просто использовать List, т.к. данная коллекция не является иммутабельной, потому что её можно закастить к MutableList. 

Чтобы решить эту проблему, был введен stong skipping mode, который работает на уровне компилятора. Он вводит два новых правила: 
1) пропуск нестабильных параметров путем сравнения по ссылке 
2) lambda memoization или автоматическое оборачивание лямб в `remember { }` 

Strong skipping mode автоматически включен с compose 1.7 или его можно включить вручную (с 1.5.4) через: 

```kotlin 
composeCompiler {
    enableStrongSkippingMode = true
}
```


Отмечу, что strong skipping mode не является панацеей, и не отменяет использование @Immutable. Он просто добавляет сравнение по ссылкам.

### Compose Compiler Configuration

Конфигурация позволяет пометить классы внутри пакета как стабильными без использования аннотаций. 

Настройка конфигурации: 
1) создание файла в корне проекта `compose_compiler_config.conf`
2) заполнение файла классами и пакетами которые стабильные, например: 
```kotlin
// 1. Делаем стабильными все классы в домене твоего проекта
com.missionchat.domain.models.*

// 2. Делаем стабильными классы из стандартной библиотеки Kotlin
// (Полезно, если используешь обычные List/Set/Map и берешь ответственность за их неизменяемость)
kotlin.collections.*

// 3. Стандартные классы Java/Kotlin для работы со временем
java.time.LocalDateTime
java.time.LocalDate
java.time.ZoneId

// 4. Можно указать конкретный класс, если не хочешь открывать весь пакет
com.example.thirdparty.SomeSpecificModel
```
3) настроить `composeCompiler { }` во всех gradle файлах с compose плагином: 
```kotlin
plugins {
    // ...
    id("org.jetbrains.kotlin.plugin.compose") 
}

// Конфигурация нового компилятора Compose
composeCompiler {
  //...
stabilityConfigurationFiles.add(rootProject.layout.projectDirectory.file("compose_compiler_config.conf"))
}
```
или можно применить только в корневом градл файле, только если проект не использует конвеншен плагины 
```kotlin
// Корневой build.gradle.kts
subprojects {
    plugins.withId("org.jetbrains.kotlin.plugin.compose") {
        composeCompiler {
stabilityConfigurationFiles.add(project.layout.projectDirectory.file("compose_compiler_config.conf"))
        }
    }
}
```
4) чтобы проверить, что конфигурация сработала нужно использовать Compose Compiler MetricsЮ которые включаются аналогично через gradle: 
```kotlin
composeCompiler {
	//...
	
    // Папка, куда компилятор сложит отчеты
    val metricsDir = project.layout.buildDirectory.dir("compose_metrics")
    metricsDestination.set(metricsDir)
    reportsDestination.set(metricsDir)
}
```
